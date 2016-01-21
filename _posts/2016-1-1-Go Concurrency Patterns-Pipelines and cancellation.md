---
layout: post
title: "『译』Go Concurrency Patterns: Pipelines and cancellation"
description: ""
category:
tags: ["golang"]
---

### 介绍

Go 的并发原语可以轻松的的通过建立数据流pipeline来有效的利用I/O和多CPU。本文介绍了这种pipeline的一些例子，强调了当操作失败时的细节处理，以及错误后如何清理。

### 什么是pipeline

在Go中pipeline并没有明确的定义；它只是多种并发程序中的一种。通俗的讲，一个pipeline是由一系列channel连接的_stages_，其中每个stage是一组运行相同函数的goroutine。在每个stage中，goroutine

* 通过_输入_channel从_上游_接收值
* 对这些数据执行某些函数，通常是生成一些新的值
* 通过_输出_channel发送值到_下游_

每个阶段有任意数量的输入输入_channel_，除了第一个和最后阶段，分别只有输出或输入channel。第一个阶段有时被称为_source_或_producer_；最后阶段称为_sink_或_consumer_。

我们将开始通过一个简单的pipeline的例子来阐述这个思想和技巧。稍后，我们会介绍一个更贴近现实的例子。

### 阶乘数字

思考一个有三个stage的pipeline。

第一个stage，`gen`，是一个转换一个integer的list并将list中的integer发送到一个channel中的函数。这个`gen`函数启动一个goroutine将integer发送到channel并且当所有值发送完后关闭channel：

```go
func gen(nums ...int) <-chan int {
	out := make(chan int)
	go func() {
		for _, n := range nums {
			out <- n
		}
		close(out)
	}()
	return out
}
```

第二个stage，`sq`，从一个channel中接收integer，将每个integer平方后发送到一个channel中并返回这个channel。在输入channel关闭并将这个阶段的所有值发送到下游后，关闭输出channel:

```go
func sq(in <-chan int) <-chan int {
	out := make(chan int)
	go func() {
		for n := range in {
			out <- n * n
		}
		close(out)
	}()
	return out
}
```

在`main`函数中建立pipeline并运行最后一个stage：它从第二个stage中接收值并显示出每一个，直到channel被关闭：

```go
func main() {
	// Set up the pipeline.
	c := gen(2, 3)
	out := sq(c)

	// Consume the output.
	fmt.Println(<-out) // 4
	fmt.Println(<-out) // 9
}
```

因为`sq`有相同类型的输入输出channel，我们可以任意次数的组合他们。我们还可以在`main`中重写一个范围循环，像这样的一组stage：

```go
func main() {
	// Set up the pipeline and consume the output.
	for n := range sq(sq(gen(2, 3))) {
		fmt.Println(n) // 16 then 81
	}
}
```

### Fan-out,fan-in

多个函数可以从相同的channel中读取直到该channel关闭；这就是所谓的_fan-out_。这样就提供了一种在一组workers中分配work使之并行的使用CPU和I/O的方法。

A function can read from multiple inputs and proceed until all are closed by multiplexing the input channels onto a single channel that's closed when all the inputs are closed.  This is called _fan-in_.

我们可以改变我们的pipeline来运行两个`sq`实例，每个都从相同的输入channel读取。我们采用一个新的函数，_merge_，来汇总在结果中：

```go
func main() {
	in := gen(2, 3)

	// Distribute the sq work across two goroutines that both read from in.
	c1 := sq(in)
	c2 := sq(in)

	// Consume the merged output from c1 and c2.
	for n := range merge(c1, c2) {
		fmt.Println(n) // 4 then 9, or 9 then 4
	}
}
```

这个`merge`函数通过一个goroutine来转换一个channel的list到一个channel来把每个输入channel中的值复制到唯一的输出channel中。一旦所有的`output`goroutine都启动了，`merge`启动另一个gorotine在在这个channel发送完成后关闭这个输出channel。

发送到一个已经关闭的channel会引起panic，所以重要的是要确保在关闭前都已经发送完。[`sync.WaitGroup`](http://golang.org/pkg/sync/#WaitGroup)类型提供了一种简单的方法来安排这种同步。

```go
func merge(cs ...<-chan int) <-chan int {
	var wg sync.WaitGroup // HL
	out := make(chan int)

	// Start an output goroutine for each input channel in cs.  output
	// copies values from c to out until c is closed, then calls wg.Done.
	output := func(c <-chan int) {
		for n := range c {
			out <- n
		}
		wg.Done() // HL
	}
	wg.Add(len(cs)) // HL
	for _, c := range cs {
		go output(c)
	}

	// Start a goroutine to close out once all the output goroutines are
	// done.  This must start after the wg.Add call.
	go func() {
		wg.Wait() // HL
		close(out)
	}()
	return out
}
```

### Stopping short

这是我们pipeline功能的一个模式：

* 当所有发送操作结束后stages关闭他们的输出channels。
* stages持续的从输入channels中接收值直到这些channel都关闭。

该模式允许每个接收stage都写成`range`循环，并且一旦所有的值都顺利的发送到下游将确保所有的goroutine退出。

但在实际的pipeline中，stages并不总是获取所有的输入值。经常是因为这样的设计：接收着在进行下一步只需要这些值中的子集。更多的时候，当一个stage因为一个输入的值提早结束时意味着在前面的stages中出现了一个错误。在这些情况下接收者都不用等待剩下的值的到来，并且我们希望前面的stages停止值的生产但是后面的stages并不需要这样。

在我们的pipeline例子中，如果一个stage在处理所有的输入值时发生错误，这些goroutines试图发送这些值的时候会无限期的阻塞。

```go
	// Consume the first value from output.
	out := merge(c1, c2)
	fmt.Println(<-out) // 4 or 9
	return
	// Since we didn't receive the second value from out,
	// one of the output goroutines is hung attempting to send it.
}
```

这是个资源泄露：goroutines消耗内存和运行时资源，and heap references in goroutine stacks keep data from being garbage collected. Goroutines are not garbage collected; they must exit on their own.

We need to arrange for the upstream stages of our pipeline to exit even when the downstream stages fail to receive all the inbound values. 一种方法是将输出channel改为带缓冲的。缓冲可以容纳固定数量的值；如果缓冲区中有足够的空间的话发送操作可以立即完成：

```go
c := make(chan int, 2) // buffer size 2
c <- 1  // succeeds immediately
c <- 2  // succeeds immediately
c <- 3  // blocks until another goroutine does <-c and receives 1
```

当要发送的值的数量在创建channel时是已知的话，缓冲可以简化代码。例如，我们可以重写`gen`来复制integer list到一个带缓冲的channel中并且避免产生新的goroutine：

```go
func gen(nums ...int) <-chan int {
	out := make(chan int, len(nums))
	for _, n := range nums {
		out <- n
	}
	close(out)
	return out
}
```

在我们的pipeline中返回一个阻塞的goroutine，我们或许可以考虑为`merge`中返回的输出channel增加一个缓冲：

```go
func merge(cs ...<-chan int) <-chan int {
	var wg sync.WaitGroup
	out := make(chan int, 1) // enough space for the unread inputs
	// ... the rest is unchanged ...
```

虽然这修复了程序中的阻塞的goroutine，但是这是糟糕的代码。这里选择缓冲大小为1取决于知道`merge`将要接收的值的数量和下游的stages将要消耗的值的数量。这是不靠谱的：如果我们给`gen`传了多余的值，或者下游的stage读了比较少的值，我们还是会将goroutines阻塞。

Instead, we need to provide a way for downstream stages to indicate to the senders that they will stop accepting input.

### Explicit cancellation

当`main`在没有从`out`中接收完所有值时决定结束，它必须告诉上游的stage中的goroutines，让其放弃正在试图发送的值。通过向一个叫`done`的channel发送一个值来做这件事。它发送两个值因为可能有两个要阻止的发送者：

```go
func main() {
	in := gen(2, 3)

	// Distribute the sq work across two goroutines that both read from in.
	c1 := sq(in)
	c2 := sq(in)

	// Consume the first value from output.
	done := make(chan struct{}, 2) // HL
	out := merge(done, c1, c2)
	fmt.Println(<-out) // 4 or 9

	// Tell the remaining senders we're leaving.
	done <- struct{}{} // HL
	done <- struct{}{} // HL
}
```

The sending goroutines replace their send operation with a `select` statement that proceeds either when the send on `out` happens or when they receive a value from `done`. `done`这个值的类型为空struct因为值并不重要：这是一个接收事件代表`out`应该放弃发送。这个`output`goroutines继续在输入channel上循环，`c`,所以上游stages没有阻塞。(我们将在稍后讨论如何让这个循环提前返回。)

```go
func merge(done <-chan struct{}, cs ...<-chan int) <-chan int {
	var wg sync.WaitGroup
	out := make(chan int)

	// Start an output goroutine for each input channel in cs.  output
	// copies values from c to out until c is closed or it receives a value
	// from done, then output calls wg.Done.
	output := func(c <-chan int) {
		for n := range c {
			select {
			case out <- n:
			case <-done: // HL
			}
		}
		wg.Done()
	}
	// ... the rest is unchanged ...
```

这种方法有个问题：_每个_下游的接收者需要知道上游可能阻塞的发送者的数量and arrange to signal those senders on early return. 持续追踪这些数量既繁琐又容易出错。

我们需要一种方法来告诉未知的和位置数量的goroutine去停止向下游发送值。在Go中，我们可以通过关闭channel来做到这点，因为[在一个关闭的channel上做接收操作总会立刻执行，会产生一个零值的元素。](http://golang.org/ref/spec#Receive_operator)

这意味着`main`可以简单通过关闭`done`channel来为所有发送者解除阻塞。这种关闭是一种有效的向发送者的广播。我们扩展pipeline的_每个_函数为其增加一个`done`的参数and arrange for the close to happen via a `defer` statement, so that all return paths from `main` will signal the pipeline stages to exit.

```go
func main() {
	// Set up a done channel that's shared by the whole pipeline,
	// and close that channel when this pipeline exits, as a signal
	// for all the goroutines we started to exit.
	done := make(chan struct{}) // HL
	defer close(done)           // HL

	in := gen(done, 2, 3)

	// Distribute the sq work across two goroutines that both read from in.
	c1 := sq(done, in)
	c2 := sq(done, in)

	// Consume the first value from output.
	out := merge(done, c1, c2)
	fmt.Println(<-out) // 4 or 9

	// done will be closed by the deferred call. // HL
}
```

Each of our pipeline stages is now free to return as soon as `done` is closed. The `output` routine in `merge` can return without draining its inbound channel, since it knows the upstream sender, `sq`, will stop attempting to send when `done` is closed.  `output` ensures `wg.Done` is called on all return paths via a `defer` statement:

```go
func merge(done <-chan struct{}, cs ...<-chan int) <-chan int {
	var wg sync.WaitGroup
	out := make(chan int)

	// Start an output goroutine for each input channel in cs.  output
	// copies values from c to out until c or done is closed, then calls
	// wg.Done.
	output := func(c <-chan int) {
		defer wg.Done() // HL
		for n := range c {
			select {
			case out <- n:
			case <-done:
				return // HL
			}
		}
	}
	// ... the rest is unchanged ...
```

Similarly, `sq` can return as soon as `done` is closed.  `sq` ensures its `out` channel is closed on all return paths via a `defer` statement:

```go
func sq(done <-chan struct{}, in <-chan int) <-chan int {
	out := make(chan int)
	go func() {
		defer close(out) // HL
		for n := range in {
			select {
			case out <- n * n:
			case <-done:
				return // HL
			}
		}
	}()
	return out
}
```

Here are the guidelines for pipeline construction:

* stages close their outbound channels when all the send operations are done.
* stages keep receiving values from inbound channels until those channels are closed or the senders are unblocked.

Pipelines unblock senders either by ensuring there's enough buffer for all the values that are sent or by explicitly signalling senders when the receiver may abandon the channel.

### Digesting a tree

让我们考虑一个更加现实的pipeline。

MD5是一个对文件校验优秀的信息摘要算法。该命令行实用程序`md5sum`可以打印出文件列表中的摘要值。

```
% md5sum *.go
d47c2bbc28298ca9befdfbc5d3aa4e65  bounded.go
ee869afd31f83cbb2d10ee81b2b831dc  parallel.go
b88175e65fdcbc01ac08aaf1fd9b5e96  serial.go
```

我们的这个示例程序就像是`md5sum`但是替换为一个单独的目录作为参数并打印出该目录下的每个文件的摘要值，按路径名排序。

```
% go run serial.go .
d47c2bbc28298ca9befdfbc5d3aa4e65  bounded.go
ee869afd31f83cbb2d10ee81b2b831dc  parallel.go
b88175e65fdcbc01ac08aaf1fd9b5e96  serial.go
```

我们程序中的主函数调用一个辅助函数`MD5All`，返回一个路径名和摘要值的map，然后排序输出结果：

```go
func main() {
	// Calculate the MD5 sum of all files under the specified directory,
	// then print the results sorted by path name.
	m, err := MD5All(os.Args[1]) // HL
	if err != nil {
		fmt.Println(err)
		return
	}
	var paths []string
	for path := range m {
		paths = append(paths, path)
	}
	sort.Strings(paths) // HL
	for _, path := range paths {
		fmt.Printf("%x  %s\n", m[path], path)
	}
}
```

这个`MD5All`函数是我们讨论的重点。在[serial.go](http://blog.golang.org/pipelines/serial.go)中，the implementation uses no concurrency and simply reads and sums each file as it walks the tree.

```go
// MD5All reads all the files in the file tree rooted at root and returns a map
// from file path to the MD5 sum of the file's contents.  If the directory walk
// fails or any read operation fails, MD5All returns an error.
func MD5All(root string) (map[string][md5.Size]byte, error) {
	m := make(map[string][md5.Size]byte)
	err := filepath.Walk(root, func(path string, info os.FileInfo, err error) error { // HL
		if err != nil {
			return err
		}
		if !info.Mode().IsRegular() {
			return nil
		}
		data, err := ioutil.ReadFile(path) // HL
		if err != nil {
			return err
		}
		m[path] = md5.Sum(data) // HL
		return nil
	})
	if err != nil {
		return nil, err
	}
	return m, nil
}
```

### Parallel digestion

在[parallel.go](http://blog.golang.org/pipelines/parallel.go)中，我们把`MD5All`分割到一个两个stage的pipeline中。在第一个stage，`sumFiles`，walks the tree，起一个新的goroutine中摘要每个文件，然后发送类型为`result`的结果到channel中：

```go
type result struct {
	path string
	sum  [md5.Size]byte
	err  error
}
```

`sumFiles`返回两个channel：一个是`results`另一个是`filepath.Walk`返回的错误。这个walk函数启动一个新的goroutine来处理每个检查的文件，然后检查`done`。如果`done`关闭了，walk则立刻停止：

```go
func sumFiles(done <-chan struct{}, root string) (<-chan result, <-chan error) {
	// For each regular file, start a goroutine that sums the file and sends
	// the result on c.  Send the result of the walk on errc.
	c := make(chan result)
	errc := make(chan error, 1)
	go func() { // HL
		var wg sync.WaitGroup
		err := filepath.Walk(root, func(path string, info os.FileInfo, err error) error {
			if err != nil {
				return err
			}
			if !info.Mode().IsRegular() {
				return nil
			}
			wg.Add(1)
			go func() { // HL
				data, err := ioutil.ReadFile(path)
				select {
				case c <- result{path, md5.Sum(data), err}: // HL
				case <-done: // HL
				}
				wg.Done()
			}()
			// Abort the walk if done is closed.
			select {
			case <-done: // HL
				return errors.New("walk canceled")
			default:
				return nil
			}
		})
		// Walk has returned, so all calls to wg.Add are done.  Start a
		// goroutine to close c once all the sends are done.
		go func() { // HL
			wg.Wait()
			close(c) // HL
		}()
		// No select needed here, since errc is buffered.
		errc <- err // HL
	}()
	return c, errc
}
```

`MD5All`从`c`中接受摘要值。`MD5All`发生错误时则提前返回，在`defer`中关闭`done`：

```go
func MD5All(root string) (map[string][md5.Size]byte, error) {
	// MD5All closes the done channel when it returns; it may do so before
	// receiving all the values from c and errc.
	done := make(chan struct{}) // HLdone
	defer close(done)           // HLdone

	c, errc := sumFiles(done, root) // HLdone

	m := make(map[string][md5.Size]byte)
	for r := range c { // HLrange
		if r.err != nil {
			return nil, r.err
		}
		m[r.path] = r.sum
	}
	if err := <-errc; err != nil {
		return nil, err
	}
	return m, nil
}
```

### Bounded parallelism

[parallel.go](http://blog.golang.org/pipelines/parallel.go)中的`MD5All`运行时会为每个文件启动一个goroutine。在一个有一些大型文件的目录中，this may allocate more memory than is available on the machine.

我们可以限制分配的并行读取文件的数量。在[bounded.go](http://blog.golang.org/pipelines/bounded.go)中，我们通过创建固定数量的goroutine来读取文件。现在我们的pipeline有着三个stage：walk the tree，读取文件并摘要，汇总摘要。第一个stage，`walkFiles`，emits the paths of regular files in the tree:

```go
func walkFiles(done <-chan struct{}, root string) (<-chan string, <-chan error) {
	paths := make(chan string)
	errc := make(chan error, 1)
	go func() { // HL
		// Close the paths channel after Walk returns.
		defer close(paths) // HL
		// No select needed for this send, since errc is buffered.
		errc <- filepath.Walk(root, func(path string, info os.FileInfo, err error) error { // HL
			if err != nil {
				return err
			}
			if !info.Mode().IsRegular() {
				return nil
			}
			select {
			case paths <- path: // HL
			case <-done: // HL
				return errors.New("walk canceled")
			}
			return nil
		})
	}()
	return paths, errc
}
```

中间的stage启动固定数量的`digester`goroutine来从`paths`中读取文件名并且发送`results`到channel `c`中：

```go
func digester(done <-chan struct{}, paths <-chan string, c chan<- result) {
	for path := range paths { // HLpaths
		data, err := ioutil.ReadFile(path)
		select {
		case c <- result{path, md5.Sum(data), err}:
		case <-done:
			return
		}
	}
}
```

不想我们前面的例子，`digester`不会关闭输出channel，作为多个goroutines发送的共享channel。相反的，`MD5All`的代码当所有`digesters`完成时分配的channel将会关闭：

```go
// Start a fixed number of goroutines to read and digest files.
	c := make(chan result) // HLc
	var wg sync.WaitGroup
	const numDigesters = 20
	wg.Add(numDigesters)
	for i := 0; i < numDigesters; i++ {
		go func() {
			digester(done, paths, c) // HLc
			wg.Done()
		}()
	}
	go func() {
		wg.Wait()
		close(c) // HLc
	}()
```

我们可以把每个digester改成创建并返回自己的输出channel，但是我们需要添加工多的goroutine来fan-in结果。

在最后一个stage在从`errc`检查错误时从`c`中接收所有的`results`。这个检查不能过早进行，因为在这之前，`walkFiles`可能会阻塞发送到下游的值：

```go
	m := make(map[string][md5.Size]byte)
	for r := range c {
		if r.err != nil {
			return nil, r.err
		}
		m[r.path] = r.sum
	}
	// Check whether the Walk failed.
	if err := <-errc; err != nil { // HLerrc
		return nil, err
	}
	return m, nil
}
```

### Conclusion

本文介绍了一种在Go中构建streaming data pipelines的方法。在这种pipeline中处理错误是很麻烦的，因为pipeline中每个stage都可能试图阻止向下游发送值，下游也可能不再关心传入的数据。我们展示了如何通过向所有goroutine广播"done"信号来关闭channel by a pipeline and defined guidelines for constructing pipelines correctly.

Further reading:

* [Go Concurrency Patterns](http://talks.golang.org/2012/concurrency.slide#1)([video](https://www.youtube.com/watch?v=f6kdp27TYZs))presents the basics of Go's concurrency primitives and several ways to apply them.
* [Advanced Go Concurrency Patterns](http://blog.golang.org/advanced-go-concurrency-patterns) ([video](http://www.youtube.com/watch?v=QDDwwePbDtw)) covers more complex uses of Go's primitives, especially `select`.
* Douglas McIlroy's paper [Squinting at Power Series](http://swtch.com/~rsc/thread/squint.pdf) shows how Go-like concurrency provides elegant support for complex calculations.
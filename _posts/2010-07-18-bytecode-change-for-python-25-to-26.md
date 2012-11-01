---
layout: post
title: "python2.5到2.6字节码指令变化点点发现(想到什么再写什么)"
description: ""
category: "python"
tags: ["pyhtnon", "字节码"]
---
{% include JB/setup %}

《Python源码剖析》中用的Python版本为2.5.2

书中所写对

d = {"1":1, "2":2}字典创建语句所得到的字节码为(略有删减):

{% highlight python linenos %}
BUILD_MAP

DUP_TOP

LOAD_CONST (1)

ROT_TWO

LOAD_CONST ('1')

STORE_SUBSCR

DUP_TOP

LOAD_CONST (2)

ROT_TWO

LOAD_CONST ('2')

STORE_SUBSCR

STORE_NAME (d)
{% endhighlight %}

其中会有对此对栈顶字典元素的重复创建和栈顶两元素的对调



而在我正在使用的2.6.5版本中 该语句的字节码已变为(略有删减):

{% highlight python linenos %}
BUILD_MAP

LOAD_CONST (1)

LOAD_CONST ('1')

STORE_MAP           

LOAD_CONST (2)

LOAD_CONST ('2')

STORE_MAP           

STORE_NAME (d)
{% endhighlight %}

操作已经简化为了直接取出两元素作为键和值插入到字典中

同时STORE_SUBSCR的功能也有了相应的变化

嚓

………………

……………

…………

………

……

…

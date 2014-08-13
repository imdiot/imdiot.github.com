---
layout: post
title: "Linux+SkyEye+μC/OS II"
description: ""
category:
tags: ["linux", "skyeye", "uc/os"]
---

这学期操作系统课设

老师给了一套大概是《嵌入式实时操作系统μC/OS-II》光盘里的东西 什么BC3的编译器啊什么的……

结果我口就贱了一下 跟老师说我在Linux里弄吧 上机就不去了 到时候把本拿过来给您检查

然后才发现给自己找了个麻烦差事

自己根本就没接触过这些东西嘛…………

先是装SkyEye

AUR->搜->有->1.2.8版本->装->装不上

自己编译 www.skyeye.org被墙 挂上gappproxy

先是最新版的skyeye1.3.0 我的gcc4.4.3编译不了 google之 无解 自己研究之 无解 又懒得在装个gcc3

接者skyeye1.2.9 不行……

继续1.2.8 OK了~ gcc4.4.3编译过去了~

然后装交叉编译器 pacman一下 发现还真有个cross-arm-elf-binutils、cross-arm-elf-gcc-base 速度装上~

然后又下了个ucos4skyeye1.9

发现用cross-arm-elf-binutils、cross-arm-elf-gcc-base编译不了

又下了个arm-elf-tools-20030314.sh

安装出现
{% highlight bash %}
tail: 无法打开"+43" 读取数据: 没有那个文件或目录

gzip: stdin: not in gzip format
tar: 它似乎不像是一个 tar 归档文件
tar: 由于前次错误，将以上次的错误状态退出
{% endhighlight %}
打开arm-elf-tools-20030314.sh 将tail后加入-n 并在文件末尾添加一空行 交叉编译器OK

make config 出现：
{% highlight bash %}
building target platform is /bin/sh: line 0: test: =: unary operator expected
/bin/sh: line 4: test: =: unary operator expected
 ???
 ------------------------------------------  
 I can not guess the host operation system
 please set OSTYPE variable in rules.make !
 or execute command export OSTYPE=linux-gnu 
 in bash shell, if your host system is linux.
 or execute commands export OSTYPE=cygwin 
 in cygwin bash shell, if your host system is cgwin.
 Then you should try make config again!
 ------------------------------------------ 
{% endhighlight %}
在samples/dir.make中添加OSTYPE=linux-gnu

make config

make dep

进入samples/simple_test

make 继续error……
{% highlight bash %}
skyeye_printf.o: In function `getnum':
/home/yangguang/Src/ucosii4skyeye/samples/simple_test/../../lib/skyeye_printf.c:110: undefined reference to `isdigit'
skyeye_printf.o: In function `skyeye_printf':
/home/yangguang/Src/ucosii4skyeye/samples/simple_test/../../lib/skyeye_printf.c:156: undefined reference to `isdigit'
make: *** [simple_test.elf] 错误 1
{% endhighlight %}
改之 在lib/skyeye_printf.c添加
{% highlight c %}
static int isdigit(char ch)
{
    if (ch >= '0' && ch <= '9')
        return 1;
    return 0;
}
{% endhighlight %}
并注释掉`#include <ctype.h>`

根据readme输入`skyeye -e simple_test.elf`

发现n多error开始刷屏

继续google 发现要`skyeye -c skyeye.conf -e simple_test.elf`

终于能仿真了 却发现都是乱码……直接悲剧……

尝试了半天 终于在重新编译skyeye1.2.6后才一切正常

当场泪流满面……

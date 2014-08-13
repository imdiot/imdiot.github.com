---
layout: post
title: "有关python的反编译"
description: ""
category:
tags: ["python", "反编译"]
---

最近把买了一直还没看的《Python源码剖析》看了看 获益颇多 直接导致了对python的反编译产生了不小的兴趣

google一番后找到了两份可供参考的代码： 1、[decompyle](http://www.crazy-compilers.com/) 2、[decompile](http://users.cs.cf.ac.uk/J.P.Giddy/python/decompiler/decompiler.html)



1 [decompyle](http://www.crazy-compilers.com/)是传说中反编译效果最好的 但已经成为了商业软件 目前在网上可以找到的只支持到了python2.3 但听说成为商业软件后好像也没赚多少钱…… 不过却有不少基于开源版[decompyle](http://www.crazy-compilers.com/)的修改版 试了试 在google code上的一个unpyc是一个既支持python2.6且能跑的起来的

2 [decompile](http://users.cs.cf.ac.uk/J.P.Giddy/python/decompiler/decompiler.html)好像就不怎么有名了……而且我也没有成功的跑起来……



看了看这两个的代码

1 [decompyle](http://www.crazy-compilers.com/)用的是编译原理的方法 用的是Spark解析器/编译器框架 通过Spark来定义python bytecode的句法规则、构建AST树、代码生成，在pyc->bytecode时可能是因为那时python还没有dis这个模块 所以用的是手工都取分析的方法……

2 [decompile](http://users.cs.cf.ac.uk/J.P.Giddy/python/decompiler/decompiler.html)则是借鉴了python虚拟机执行时的方法 python虚拟机通过bytecode来决定当前该做怎样的动作 根据bytecode所对应的虚拟机的操作来生成反编译代码



1 [decompyle](http://www.crazy-compilers.com/)的方法思路清晰 但对Spark的理解比较费事……而且对编译原理的知识要求的比较多……如果没学过编译原理的话基本是一头雾水…… 但想修改、扩充起来可能会方便的多

2 [decompile](http://users.cs.cf.ac.uk/J.P.Giddy/python/decompiler/decompiler.html)的方法……简单……很容易理解 除了简单好像就没什么了……



PS:拿python的反编译做毕设会不会比较有趣呢…………

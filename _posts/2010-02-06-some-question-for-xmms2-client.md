---
layout: post
title: "有关写xmms2 client的一个很奇怪的问题"
description: ""
category: "python"
tags: ["python", "xmms2"]
---
{% include JB/setup %}

在初始化好xmmsclient.XMMS实例后

要先用xmmsclient.XMMS.connect()来连接XMMS2

但因为此时XMMS2并未运行 所以会产生一个错误

try…… except……到此错误后

先用`glib.spawn_async(["/usr/bin/xmms2-launcher", "--yes-run-as-root"])`来启动XMMS2

再继续用xmmsclient.XMMS.connect()来连接XMMS2

此时 一个奇怪的问题发生了

经测试 大概在启动XMMS2的0.1秒左右的时间内总是无法connect到XMMS2

但只要大于这个时间就可以连接上

百思不得其解 难道说是因为connect的时候XMMS2进程还没有初始化完毕???

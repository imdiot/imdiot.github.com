---
layout: post
title: "Firefly and Pomelo"
description: ""
category:
tags: ["firefly", "pomelo"]
---

最近几年做手游后端一直用的都是各种webserver，太过民工了，准备高大上一把研究研究socketserver。

放狗搜了一下，当然是python的，有个9秒的firefly，不过粗粗看了几眼感觉是比较粗糙。

firefly有两个版本，一个是基于twisted的，一个是基于gevent的，看趋势应该未来是以gevent版本为主的。

gevent的版本为了便于迁移，9miao又为gevent做了一个gtwisted的封装，所以gfirefly的各种东西还是和firefly差不多的，不过因为多了gtwisted这一层感觉很丑啊。

db层就不多说了，看了两眼应该是草草做的，够用但不好看。

不过想起以前看到网易出的Pomelo，虽然是node.js但，但可以参考的嘛，继续草草一看，哇擦，代码各种整洁高大上，还有wiki看。

就它了，这几天好好研读一下Pomelo。

---
layout: post
title: "加速yaourt----yaourt与makepkg调用其他下载工具"
description: ""
category:
tags: ["archlinux", "yaourt", "makepkg"]
---

各位用archlinux的朋友们都知道pacman能够调用外部下载工具来代替默认的wget来给pacman提速

比如将`/etc/pacman.conf中XferCommand = /usr/bin/wget --passive-ftp -c -O %o %u`注释掉

同时添加`XferCommand = /usr/bin/axel -o %o %u`



但arch中还有一个有力的工具yaourt，用它可以方便的从AUR上安装软件

但绝大多数情况下用其默认的wget下载时还是会觉得很慢

所以也很想把yaourt弄成多线程下载的

最开始翻了半天的yaourt源代码 怎么也找不到

快要放弃时才发现原来yaourt是调用makepkg来实现下载、编译、安装的

一下子就明了了

在`/etc/makepkg.conf`中有
{% highlight bash %}
DLAGENTS=('ftp::/usr/bin/wget -c --passive-ftp -t 3 --waitretry=3 -O %o %u'
          'http::/usr/bin/wget -c -t 3 --waitretry=3 -O %o %u'
          'http::/usr/bin/axel -o %o %u'
          'https::/usr/bin/wget -c -t 3 --waitretry=3 --no-check-certificate -O %o %u'
          'rsync::/usr/bin/rsync -z %u %o'
          'scp::/usr/bin/scp -C %u %o')
{% endhighlight %}
表明了各种协议所调用的下载工具

一般下载都是http的

所以我将`http::/usr/bin/wget -c -t 3 --waitretry=3 -O %o %u`改成了`http::/usr/bin/axel -o %o %u`



至此一切OK,来体验一下yaourt以及makepkg多线程的下载速度吧~

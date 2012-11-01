---
layout: post
title: "小hack一下gnome-shell的alt+tab"
description: ""
category: "linux"
tags: ["linux", "gnome-shell"]
---
{% include JB/setup %}

[TualatriX](http://imtx.me/)在《[GNOME Shell的“Alt＋Tab”革新](http://imtx.me/archives/1500.html)》中介绍了gnome-shell的alt+tab 那时我还没用过gnome-shell 虽说之后偶尔耍耍gnome-shell 但也是浅尝即止 这几天有时间好好玩了玩gnome-sehll

看过TualatriX的那篇文章后 发现自己的alt+tab的快捷键与那时的已经不一样了 可能是代码做了调整了 已经没有alt+`了 alt+a、w、s、d也变成了alt+上、下、左、右 感觉还没有`、a、w、s、d方便

所以就hack了一下alt+tab 具体的使用方式可以看TualatriX的文章

{% highlight diff linenos %}
[yangguang] [~/gnome-shell/install/share/gnome-shell/js/ui]
> diff altTab.js ../../../../../source/gnome-shell/js/ui/altTab.js
253c253
<             else if (keysym == Clutter.Left || keysym == Clutter.a)
---
>             else if (keysym == Clutter.Left)
255c255
<             else if (keysym == Clutter.Right || keysym == Clutter.d || keysym == 96)
---
>             else if (keysym == Clutter.Right)
257c257
<             else if (keysym == Clutter.Up || keysym == Clutter.w)
---
>             else if (keysym == Clutter.Up)2
62c262
<             else if (keysym == Clutter.Left || keysym == Clutter.a)
---
>             else if (keysym == Clutter.Left)
264c264
<             else if (keysym == Clutter.Right || keysym == Clutter.d)
---
>             else if (keysym == Clutter.Right)
266c266
<             else if (keysym == Clutter.Down || keysym == Clutter.s || keysym == 96)
---
>             else if (keysym == Clutter.Down)
{% endhighlight %}

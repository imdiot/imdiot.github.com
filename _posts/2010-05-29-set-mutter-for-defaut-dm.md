---
layout: post
title: "将mutter作为默认窗口管理器"
description: ""
category: "linux"
tags: ["linux", "mutt", "dm"]
---
{% include JB/setup %}

对于我这种喜欢终端透明等一点点的特效 又不喜compiz那庞大体积的人来说 mutter确实是一个比较不错的选择

通过`gconftool-2 --set /desktop/gnome/session/required_components/windowmanager --type string mutter`将mutter设为默认窗口管理器

或用`gconf-editor将/desktop/gnome/session/required_components/windowmanager`改为mutter

重启X即可

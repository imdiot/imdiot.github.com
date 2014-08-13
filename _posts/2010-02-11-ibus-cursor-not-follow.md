---
layout: post
title: "对ibus输入法不跟随下了次狠手"
description: ""
category:
tags: ["ibus-sunpinyin"]
---

以前我的Arch中的ibus输入框就一直不跟随

忘了google到什么改过后就正常了

昨天一次大升级 更新了好几百Ｍ的东西

今天一看ibus又不跟随了

我也懒的再google了

干脆下狠手

把`/etc/gtk-2.0/gtk.immodules`中所有不用的全部注释掉 只留ibus

至于以后万一要换别的输入法的话…………

以后的事以后再说吧…………

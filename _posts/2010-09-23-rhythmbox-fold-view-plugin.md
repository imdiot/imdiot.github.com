---
layout: post
title: "Rhythmbox文件夹视图浏览的插件"
description: ""
category:
tags: ["rhythmbox", "plugin"]
---

#介绍

Rhythmbox浏览音乐时只能按照音乐文件的信息来分类

但有时有许多音乐文件的信息并不完整且音乐文件又很多的时候 按照音乐信息分类就不如直接按照目录分类清晰了

所以就写了一个文件夹视图的插件

文件夹视图的根目录是读取自rhythmbox首选项中的音乐文件存放位置的

#项目地址

[http://code.google.com/p/folderview/](http://code.google.com/p/folderview/)

#截图

<img src="/images/2010-09-23-rhythmbox-fold-view-plugin.png" width="100%" />

#安装

svn checkout http://folderview.googlecode.com/svn/trunk/ folderview

cp -R folderview/FolderView ~/.gnome2/rhythmbox/plugins/ 或 cp -R folderview/FolderView /usr/lib/rhythmbox/plugins/

#使用

启动Rhythmbox

菜单栏 编辑->插件

勾选“文件夹视图”

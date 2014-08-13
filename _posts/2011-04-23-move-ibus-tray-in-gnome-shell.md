---
layout: post
title: "来~让我们把gnome-shell中ibus的tray挪挪地儿~~~"
description: ""
category:
tags: ["linux", "gnome-shell", "ibus"]
---

在很久很久以前 那时的gnome-shell的系统托盘还在右上角

不知从何时起 他们分成了status-icon和message-icon

status-icon有了CSS的外衣住在了右上角 message-icon则搬到了右下角

但是因为底部的通知区域设计成了有消息才会有反应 例如dbus的一些通知什么的

因此产生了一些不爽的感觉

比如我想看一些程序tray的变化 我只能把鼠标挪到右下角才能看到

毕竟现在适合gnome-shell的这种通知模式的程序还很少

下面就以ibus为例 让ibus的tray挪挪窝：

<img src="/images/2011-04-23-move-ibus-tray-in-gnome-shell-1.png" width="100%" />

打开statusIconDispatcher.js文件

我用的是arch+testing安装的gnome-shell，statusIconDispatcher.js文件在`/usr/share/gnome-shell/js/ui`目录下

如果是自己编译的gnome-shell那就应该在`~/gnome-shell/install/share/gnome-shell/js/ui`目录下

看到`statusIconDispatcher.js`中的`STANDARD_TRAY_ICON_IMPLEMENTATIONS`变量了吗 没错 这就是右上方status-icon的白名单~

这个字典的key呢 则是程序tray的wm_class

大家看出来了吧 其实gnome-shell是想把ibus的tray放在右上方的status-icon区域的

可他们毕竟不是中国人 不用ibus………… 这个ibus-ui-gtk好像还是很久很久以前要装ibus-gtk、ibus-qt时候的东西呢吧？？？？

现在ibus tray的wm_class没有设 是默认的main.py……

所以我们只要吧ibus-ui-gtk改成main.py就大功告成啦~~~

<img src="/images/2011-04-23-move-ibus-tray-in-gnome-shell-2.png" width="100%" />

其实这根本就不是什么问题嘛 只要ibus的人和gnome-shell的人稍微沟通一些 下个版本大家一起做一两行的就该就OK了嘛

白名单加上statusIconDispatcher.js中的\_onTrayIconAdded、\_onTrayIconRemoved就可以随你挪啦~想挪谁挪谁 像以前那样放在一起都没问题~

<img src="/assets/images/2011-04-23-move-ibus-tray-in-gnome-shell-3.png" width="100%" />

PS:

ibus extensions

{% highlight js %}
StatusIconDispatcher = imports.ui.statusIconDispatcher;

function main() {
    StatusIconDispatcher.STANDARD_TRAY_ICON_IMPLEMENTATIONS['main.py'] = 'ibus';
}
{% endhighlight %}

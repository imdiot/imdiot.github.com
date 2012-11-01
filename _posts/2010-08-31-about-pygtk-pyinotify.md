---
layout: post
title: "关于PyGTK+pyinotify"
description: ""
category: "python"
tags: ["python", "pyinotify"]
---
{% include JB/setup %}

首先是pyinotify官网上的一个例子

{% highlight python linenos %}
import pyinotify

wm = pyinotify.WatchManager()  # Watch Manager
mask = pyinotify.IN_DELETE | pyinotify.IN_CREATE  # watched events

class EventHandler(pyinotify.ProcessEvent):
    def process_IN_CREATE(self, event):
        print "Creating:", event.pathname

    def process_IN_DELETE(self, event):
        print "Removing:", event.pathname

handler = EventHandler()
notifier = pyinotify.Notifier(wm, handler)
wdd = wm.add_watch('/tmp', mask, rec=True)

notifier.loop()
{% endhighlight %}

在这个例子中最后一行的notifier.loop()是inotify自己的loop实现

但如果想要PyGTK中使用inotify的话notifify的loop会和GTK的loop发生冲突

所以要做下小的修改

{% highlight python linenos %}
import pyinotify
import gobject
import gtk

gobject.threads_init()

wm = pyinotify.WatchManager()  # Watch Manager
mask = pyinotify.IN_DELETE | pyinotify.IN_CREATE  # watched events

class EventHandler(pyinotify.ProcessEvent):
    def process_IN_CREATE(self, event):
        print "Creating:", event.pathname

    def process_IN_DELETE(self, event):
        print "Removing:", event.pathname


def _quit(widget):
    notifier.stop()
    gtk.main_quit()

handler = EventHandler()
notifier = pyinotify.ThreadedNotifier(wm, handler)
wdd = wm.add_watch('/tmp', mask, rec=True)
notifier.start()

win=gtk.Window()
win.connect("destroy",_quit)
win.show()
gtk.main()
{% endhighlight %}

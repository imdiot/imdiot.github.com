---
layout: post
title: "简单的gnome自动换壁纸"
description: ""
category: "python"
tags: ["python", "gnome", "linux", "壁纸"]
---
{% include JB/setup %}

闲的没事写着玩玩

{% highlight python linenos %}
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import gconf
import os
import gobject
import gtk

class Auto_wp:
    def __init__(self, wp_dir):
        self.wp_dir     = wp_dir
        self.file_list  = []
        self.index      = 0
        self.client     = gconf.client_get_default()
        self.get_file_list(self.wp_dir)
        gobject.timeout_add(10000, self.change_wp)
        gtk.main()

    def get_file_list(self, wp_dir):
        for parent, dirnames, filenames in os.walk(wp_dir):
            for filename in filenames:
                self.file_list.append(os.path.join(parent, filename))

    def change_wp(self):
        print self.file_list[self.index]
        self.client.set_string('/desktop/gnome/background/picture_filename',self.file_list[self.index])
        self.index += 1
        if self.index >= len(self.file_list):
            self.index = 0
        gobject.timeout_add(10000, self.change_wp)

if __name__ == "__main__":
    Auto_wp('/home/yangguang/图片/background/down')
{% endhighlight %}

---
layout: post
title: "查看memcache所有key的小程序"
description: ""
category:
tags: ["python", "memcache"]
---

本地开发过程中经常要查看内容写没写到memcache中，一般看一下memcache中有没有想对应的key就好了，懒得用telnet连接memcache，自带的memcached-tool输出又不怎么方便阅读，so 写了一个简单的查看memcache中key的python小程序

{% highlight python linenos %}
# /usr/bin/env python
# -*- coding: utf-8 -* 
import socket
import re

class MemcacheServer(object):
    def __init__(self):
        self.server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

    def connect(self, host=None, port=None):
        self.host = host
        self.port = port
        try:
            self.server.connect((host, port))
        except:
            print "connect error ......"


    def send(self, msg):
        self.server.send(msg)

    def get_msg(self):
        buf_len = 1024
        msg = u""
        while True:
            buf = self.server.recv(buf_len)
            msg += buf
            if len(buf)!=buf_len:
                break
        return msg

if __name__ == "__main__":
    ms = MemcacheServer()
    ms.connect("127.0.0.1", 11211)

    items = {}
    totalitems = 0
    ms.send("stats items\r\n")
    for line in ms.get_msg().splitlines():
        match = re.search(u"^STAT items:(\d*):number (\d*)", line)
        if match:
            i, j = match.groups()
            items[int(i)] = int(j)
            totalitems += int(j)

    for buckets in sorted(items.keys()):
        ms.send("stats cachedump %d %d\r\n" % (buckets, items[buckets]))
        for line in ms.get_msg().splitlines():
            match = re.search(u"^ITEM (\S+) \[.* (\d+) s\]", line)
            if match:
                print match.groups()[0]
{% endhighlight %}

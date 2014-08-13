---
layout: post
title: "获得pyc文件的magic数字和创建时间"
description: ""
category:
tags: ["python"]
---

{% highlight python linenos %}
import struct
import time
import marshal
file = open(file_name,'rb')
str_magic = file.read(4)
    str_time = file.read(4)
    code = marshal.load(file)
    file.close()
    
    pyc_magic   = struct.unpack('i',str_magic)[0] & (~0xffff0000)
    pyc_time    = time.ctime(struct.unpack('i',str_time)[0])
    print pyc_magic,pyc_time
{% endhighlight %}
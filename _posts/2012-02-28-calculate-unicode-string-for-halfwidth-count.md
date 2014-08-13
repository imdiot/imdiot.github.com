---
layout: post
title: "python计算unicode字符转折合半角字符数"
comments: true
description: ""
category:
tags: ["python"]
---

{% highlight python linenos %}
import unicodedata

def get_unicode_halfwidth(ustr):
    if type(ustr) != unicode:
        raise TypeError, "not a unicode string"

        halfwidth = 0
        for uchar in ustr:
           width_tag = unicodedata.east_asian_width(uchar)
        halfwidth += 2 if (width_tag == 'W') or (width_tag == 'F') else 1
    return halfwidth
{% endhighlight %}

---
layout: post
title: "PySVN checkout时中文路径的解决"
description: ""
category:
tags: ["python", "pysvn"]
---

在玩PySVN时发现如果文件路径出现中文的话会出现"Can't convert string from 'UTF-8' to native encoding:"错误


祭出谷歌娘，发现早在05年就已经有了解决方法了

{% highlight python linenos %}
import locale
language_code, encoding = locale.getdefaultlocale()
if language_code is None:
    language_code = 'en_GB'
if encoding is None:
    encoding = 'UTF-8'
if encoding.lower() == 'utf':
    encoding = 'UTF-8'
locale.setlocale( locale.LC_ALL, '%s.%s' % (language_code, encoding))
{% endhighlight %}

---
layout: post
title: "Pomelo 源码阅读（四）（PushScheduler组件）"
description: ""
category:
tags: ["pomelo"]
---

首先是wiki的Connection组件介绍:

>pushScheduler组件也是一个功能较为简单的组件，它仅仅被前端服务器加载，与connector组件的关系密切。当connector组件收到server组件的对客户端请求的响应后，connector并不直接将此响应返回给客户端，而是将这个给客户端发送响应的操作调度给scheduler组件。pushScheduler组件完成最后通过session组件拿到具体的客户端连接并将请求的响应发送给客户端的任务。因此，通过pushScheduler组件可以对发给用户的响应进行缓冲，从而提高通信效率。pomelo实现了两种调度策略，一种是不进行任何缓冲，直接将响应发送给客户端，一种是进行缓冲，并定时地将已缓冲的响应发送给对应的客户端。

>pushScheduler配置项:

>   * scheduler： scheduler组件的具体调度策略配置，默认的是直接将响应发给客户端，同时pomelo还提供了有缓冲并且定时刷新的调度策略。用户也可以自定义自己的调度策略。

>配置`pushScheduler`组件，通过调用如下:

>`app.set('pushSchedulerConfig', opts);`

>如果要启用使用缓冲的scheduler的话，可以在app.js中增加:

>`app.set('pushSchedulerConfig', {scheduler: pomelo.pushSchedulers.buffer, flushInterval: 20});`

>flushInterval是刷新周期，默认为20毫秒。  

`pomelo/lib/components/pushScheduler.js`

`pushScheduler`有两种调度策略，根据配置来选择：

* `direct`scheduler: 无缓冲，直接发送
* `buffer`scheduler: 缓冲发送

pushScheduler除了start、stop外，只有一个可调用的接口`schedule`，通过此接口去调用scheduler中的`schedule`。

# Direct

`pomelo/lib/pushSchedulers/direct.js`

```javascript
Service.prototype.schedule = function(reqId, route, msg, recvs, opts, cb) {
  opts = opts || {};
  if(opts.type === 'broadcast') {
    doBroadcast(this, msg, opts.userOptions);
  } else {
    doBatchPush(this, msg, recvs);
  }

  if(cb) {
    process.nextTick(function() {
      utils.invokeCallback(cb);
    });
  }
};
```

通过参数选择:

* `doBroadcast`:  广播, 通过channel来选择发送目标
* `doBatchPush`:  批量发送

# Buffer

`pomelo/lib/pushSchedulers/buffer.js`

```javascript
var Service = function(app, opts) {
  if (!(this instanceof Service)) {
    return new Service(app, opts);
  }

  opts = opts || {};
  this.app = app;
  this.flushInterval = opts.flushInterval || DEFAULT_FLUSH_INTERVAL;
  this.sessions = {};   // sid -> msg queue
  this.tid = null;
};
```

属性:

* `app`: 对应的`application`
* `flushInterval`: 刷新周期
* `sessions`: 要发送数据对应的session
* `tid`: 定时器id

buffer和direct执行是的区别是：

* 在buffer中，doBroadcast和doBatchPush中不直接发送数据
* 通过`enqueue`方法将发送信息存储在session中
* 通过定时器执行`flush`方法发送信息


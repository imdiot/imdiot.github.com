---
layout: post
title: "Pomelo 源码阅读（Session组件）（一）"
description: ""
category:
tags: ["pomelo"]
---

首先是wiki的session组件介绍:

>session组件跟connector相关，也是仅仅被前端服务器加载，为sessionService提供一个组件包装, 加载session组件后，会在app的上下文中增加sessionService，可以通过app.get('sessionService')获取。它主要用来维护客户端的连接信息，以及生成session并维护session。如果与经典TCP进行类比的话，那么session中维护的连接就可以粗略地认为就是TCP服务器端accept返回的socket句柄。一个连接与一个session对应，同时session组件还维护具体登录用户与session的绑定信息。一个用户可以有多个客户端登录，对应于多个session。当需要给客户端推送消息或者给客户端返回响应的话，必须通过session组件拿到具体的客户端连接来进行。

>session组件支持如下配置项:

>   * singleSession： 如果这个配置项配置为true的话，那么将将不允许一个用户同时绑定到多个session，在绑定用户一次后，后面的绑定将会失败。

>配置session组件，通过调用如下方式进行:
>app.set('sessionConfig', opts);

`pomelo/lib/components/session.js`加载实际的sessionService:`pomelo/common/service/sessionService.js`

# Session

`pomelo/common/service/sessionService.js`

```javascript
var Session = function(sid, frontendId, socket, service) {
  EventEmitter.call(this);
  this.id = sid;          // r
  this.frontendId = frontendId; // r
  this.uid = null;        // r
  this.settings = {};

  // private
  this.__socket__ = socket;
  this.__sessionService__ = service;
  this.__state__ = ST_INITED;
};
```

Session属性:

* `id`: socket id //socket.id

* `frontendId`: 前端服务器id

* `uid`: 用户id

* `settings`: 设置

* `__socket__`: session对应的socket

* `__sessionService__`: sessionService

* `__state__`: session状态

```javascript
Session.prototype.bind = function(uid) {
  this.uid = uid;
  this.emit('bind', uid);
};
```

绑定用户id，触发`bind`事件。

```javascript
Session.prototype.unbind = function(uid) {
  this.uid = null;
  this.emit('unbind', uid);
};
```

解除绑定用户id，触发`unbind`事件。

```javascript
Session.prototype.closed = function(reason) {
  logger.debug('session on [%s] is closed with session id: %s', this.frontendId, this.id);
  if(this.__state__ === ST_CLOSED) {
    return;
  }
  this.__state__ = ST_CLOSED;
  this.__sessionService__.remove(this.id);
  this.emit('closed', this.toFrontendSession(), reason);
  this.__socket__.emit('closing', reason);

  var self = this;
  // give a chance to send disconnect message to client

  process.nextTick(function() {
    self.__socket__.disconnect();
  });
};
```

服务器主动关闭连接， 设置session状态为关闭；从sessionService中移除session；触发session`closed`事件；触发socket`closing`事件;nextTick关闭socket连接。

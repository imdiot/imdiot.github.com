---
layout: post
title: "Pomelo 源码阅读（五）（Connector组件）"
description: ""
category:
tags: ["pomelo"]
---

首先是wiki的Connector组件介绍:

>connector组件是一个重量级的组件，它会依赖于session组件，server组件,pushScheduler组件和connection组件。connector组件仅仅被前端服务器加载，它主要用来管理客户端的连接。connector组件会加载底层的connector，创建端口监听，绑定事件响应。当有客户端连接请求时，connector组件会请求session组件，获得当前连接的session，如果session组件中没有相应的session的话，session组件会为这个新连接创建新的session，并维护相应的连接；然后connector组件还会向connection组件上报连接信息，供统计使用；最后，将拿到的session以及客户端的请求，一起抛给server组件，由server组件进行请求处理。当server组件处理完请求后，又会通过connector组件将响应返回给客户端。在返回响应给客户端的时候，connector组件做了一个缓存选择，这个缓存实现依赖于pushScheduler组件，也就是说connector组件并不是直接将相应发给客户端，而是将响应给pushScheduler组件。pushScheduler组件根据相应调度策略，可能不缓存直接通过session组件维护的连接，将响应发出去，也可能进行缓存，并按时flush。这是可以配置的。

>connector组件支持如下配置项:

>   * connector: 底层使用的通信connector，不配置的话，会默认使用sioconnector;

>   * useProtobuf: 目前仅仅支持connector配置使用hybridconnector的情况，配置其为true，将开启消息的protobuf功能；

>   * useDict： 目前仅仅支持connector配置使用hybridconnector的情况，配置其为true时，将会开启基于字典的路由消息压缩；

>   * useCrypto： 目前仅仅支持connector配置为hybridconnector的情况，配置其为true时，将会启用通信时的数字签名；

>   * encode/decode： 消息的编码解码方式，如果不配置的话，将会默认使用connector配置中，底层connector提供的相应的编码解码函数。

>   * transports：这个配置选项是用于sioconnector的，因为socket.io的通信方式可能会有多种，如websocket，xhr-polling等等。通过这个配置选项可以选择需要的方式。

>配置connector组件，通过调用如下方式进行:

>`app.set('connectorConfig', opts);`

Connector提供几种可选的通信connector:

* `hybird`:  框架自己tcp、websocket实现
* `mqtt`: 基于mqtt协议的实现
* `sio`：基于socket.io的实现
* `udp`： udp实现

如不指定，默认为socket.io的实现。

```javascript
var bindEvents = function(self, socket) {
  if(self.connection) {
    self.connection.increaseConnectionCount();
    var statisticInfo = self.connection.getStatisticsInfo();
    var curServer = self.app.getCurServer();
    if(statisticInfo.totalConnCount > curServer['max-connections']) {
      logger.warn('the server %s has reached the max connections %s', curServer.id, curServer['max-connections']);
      socket.disconnect();
      return;
    }
  }

  //create session for connection
  var session = getSession(self, socket);
  var closed = false;

  socket.on('disconnect', function() {
    if(closed) {
      return;
    }
    closed = true;
    if(self.connection) {
      self.connection.decreaseConnectionCount(session.uid);
    }
  });

  socket.on('error', function() {
    if(closed) {
      return;
    }
    closed = true;
    if(self.connection) {
      self.connection.decreaseConnectionCount(session.uid);
    }
  });

  // new message
  socket.on('message', function(msg) {
    var dmsg = msg;
    if(self.decode) {
      dmsg = self.decode(msg);
    } else if(self.connector.decode) {
      dmsg = self.connector.decode(msg);
    }
    if(!dmsg) {
      // discard invalid message
      return;
    }

    // use rsa crypto
    if(self.useCrypto) {
      var verified = verifyMessage(self, session, dmsg);
      if(!verified) {
        logger.error('fail to verify the data received from client.');
        return;
      }
    }

    handleMessage(self, session, dmsg);
  }); //on message end
};
```

监听事件，当监听到socket发出的`message`事件时执行`handleMessage`。

```javascript
var handleMessage = function(self, session, msg) {
  logger.debug('[%s] handleMessage session id: %s, msg: %j', self.app.serverId, session.id, msg);
  var type = checkServerType(msg.route);
  if(!type) {
    logger.error('invalid route string. route : %j', msg.route);
    return;
  }
  self.server.globalHandle(msg, session.toFrontendSession(), function(err, resp, opts) {
    if(resp && !msg.id) {
      logger.warn('try to response to a notify: %j', msg.route);
      return;
    }
    if (!resp) resp = {};
    if (!!err){
      resp.code = 500;
    }
    opts = {type: 'response', userOptions: opts || {}};
    // for compatiablity
    opts.isResponse = true;

    self.send(msg.id, msg.route, resp, [session.id], opts,
      function() {});
  });
};
```

这是就进入server组件的`globalHandle`方法了，等看到server组件再说。

这个写的太简单了，主要是`hybird`中事件各种绕，虽然看个半斤八两的了但是还是有点晕，至于`sio`的则是非常简单的了。以后继续写。


---
layout: post
title: "Pomelo 源码阅读（三）（Connection组件）"
description: ""
category:
tags: ["pomelo"]
---

首先是wiki的Connection组件介绍:

>connection组件是一个功能相对简单的组件，也是仅仅被前端服务器加载,为connectionService提供一个组件包装,他主要进行连接信息的统计,connector组件接收到客户端连接请求以及有客户端离线时，以及用户登录下线等等情况，都会向其汇报。
>connection组件无配置项。 

`pomelo/lib/components/connection.js`加载实际的sessionService:`pomelo/common/service/connectionService.js`

`pomelo/common/service/connectionService.js`

```javascript
var Service = function(app) {
  this.serverId = app.getServerId();
  this.connCount = 0;
  this.loginedCount = 0;
  this.logined = {};
};
```

Connection属性：

* `serverId`：服务器id
* `connCount`：连接数量
* `loginedCount`：登陆数量
* `logined`：登陆用户的uid，以及一些用户信息

```javascript
pro.addLoginedUser = function(uid, info) {
  if(!this.logined[uid]) {
    this.loginedCount++;
  }
  info.uid = uid;
  this.logined[uid] = info;
};
```

添加一个登陆用户：

* `loginedCount`登陆用户数量加一
* 绑定`uid`与`info`用户信息

```javascript
pro.increaseConnectionCount = function() {
  this.connCount++;
};
```
`connCount`连接数量减一

```javascript
pro.removeLoginedUser = function(uid) {
  if(!!this.logined[uid]) {
    this.loginedCount--;
  }
  delete this.logined[uid];
};
```

删除一个登陆用户:

* 如果用户已登陆，用户登陆数量减一
* 删除对应用户信息

```javascript
pro.decreaseConnectionCount = function(uid) {
  if(this.connCount) {
    this.connCount--;
  }
  if(!!uid) {
    this.removeLoginedUser(uid);
  }
};
```

连接数量减一:

* 连接数量减一
* 删除uid对应的登陆用户

```javascript
pro.getStatisticsInfo = function() {
  var list = [];
  for(var uid in this.logined) {
    list.push(this.logined[uid]);
  }

  return {serverId: this.serverId, totalConnCount: this.connCount, loginedCount: this.loginedCount, loginedList: list};
};
```

获得统计信息:

* `serverId`：`serverId`服务器id
* `totalConnCount`： `connCount` 连接数量
* `loginedCount`：`this.loginedCount` 登陆数量
* `loginedList`：已登陆的uid
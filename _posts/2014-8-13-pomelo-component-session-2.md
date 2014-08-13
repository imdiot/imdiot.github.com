---
layout: post
title: "Pomelo 源码阅读（Session组件）（二）"
description: ""
category:
tags: ["pomelo"]
---

# SessionService

```javascript
var SessionService = function(opts) {
  opts = opts || {};
  this.singleSession = opts.singleSession;
  this.sessions = {};     // sid -> session
  this.uidMap = {};       // uid -> sessions
};
```

SessionService属性：

* singleSession：是否允许同意用户对应多个session

* sessions: sessionId对应session

* uidMap: uid对应session


```javascript
SessionService.prototype.create = function(sid, frontendId, socket) {
  var session = new Session(sid, frontendId, socket, this);
  this.sessions[session.id] = session;

  return session;
};
``` 

新建session，并添加到sessions中

在

```javascript
SessionService.prototype.bind = function(sid, uid, cb) {
	// ***
};
```

函数中

```javascript
var session = this.sessions[sid];
```

获取sessionId对应的session

```javascript
  if(!session) {
    process.nextTick(function() {
      cb(new Error('session does not exist, sid: ' + sid));
    });
    return;
  }
```

如果session不存在发出错误信息并返回

```javascript
  if(session.uid) {
    if(session.uid === uid) {
      // already bound with the same uid
      cb();
      return;
    }

    // already bound with other uid
    process.nextTick(function() {
      cb(new Error('session has already bound with ' + session.uid));
    });
    return;
  }
```

检测session与uid是否对应

```javascript
  var sessions = this.uidMap[uid];

  if(!!this.singleSession && !!sessions) {
    process.nextTick(function() {
      cb(new Error('singleSession is enabled, and session has already bound with uid: ' + uid));
    });
    return;
  }

  if(!sessions) {
    sessions = this.uidMap[uid] = [];
  }

  for(var i=0, l=sessions.length; i<l; i++) {
    // session has binded with the uid
    if(sessions[i].id === session.id) {
      process.nextTick(cb);
      return;
    }
  }
  sessions.push(session);

  session.bind(uid);
```

将uid与session绑定。

## 未完待续



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

```javascript
SessionService.prototype.unbind = function(sid, uid, cb) {
  var session = this.sessions[sid];

  if(!session) {
    process.nextTick(function() {
      cb(new Error('session does not exist, sid: ' + sid));
    });
    return;
  }

  if(!session.uid || session.uid !== uid) {
    process.nextTick(function() {
      cb(new Error('session has not bind with ' + session.uid));
    });
    return;
  }

  var sessions = this.uidMap[uid], sess;
  if(sessions) {
    for(var i=0, l=sessions.length; i<l; i++) {
      sess = sessions[i];
      if(sess.id === sid) {
        sessions.splice(i, 1);
        break;
      }
    }

    if(sessions.length === 0) {
      delete this.uidMap[uid];
    }
  }
  session.unbind(uid);

  if(cb) {
    process.nextTick(cb);
  }
};
```

解除session和uid之间的绑定。

```javascript
SessionService.prototype.get = function(sid) {
  return this.sessions[sid];
};
```

通过sid获得session

```javascript
SessionService.prototype.getByUid = function(uid) {
  return this.uidMap[uid];
};
```

通过uid获取sessions

```javascript
SessionService.prototype.remove = function(sid) {
  var session = this.sessions[sid];
  if(session) {
    var uid = session.uid;
    delete this.sessions[session.id];

    var sessions = this.uidMap[uid];
    if(!sessions) {
      return;
    }

    for(var i=0, l=sessions.length; i<l; i++) {
      if(sessions[i].id === sid) {
        sessions.splice(i, 1);
        if(sessions.length === 0) {
          delete this.uidMap[uid];
        }
        break;
      }
    }
  }
};
```

删除一个session。

```javascript
SessionService.prototype.import = function(sid, key, value, cb) {
  var session = this.sessions[sid];
  if(!session) {
    utils.invokeCallback(cb, new Error('session does not exist, sid: ' + sid));
    return;
  }
  session.set(key, value);
  utils.invokeCallback(cb);
};
```

将一组值存入session。

```javascript
SessionService.prototype.importAll = function(sid, settings, cb) {
  var session = this.sessions[sid];
  if(!session) {
    utils.invokeCallback(cb, new Error('session does not exist, sid: ' + sid));
    return;
  }

  for(var f in settings) {
    session.set(f, settings[f]);
  }
  utils.invokeCallback(cb);
};
```

将多组值存入session

```javascript
SessionService.prototype.kick = function(uid, reason, cb) {
  // compatible for old kick(uid, cb);
  if(typeof reason === 'function') {
    cb = reason;
    reason = 'kick';
  }
  var sessions = this.getByUid(uid);

  if(sessions) {
    // notify client
    var sids = [];
    var self = this;
    sessions.forEach(function(session) {
      sids.push(session.id);
    });

    sids.forEach(function(sid) {
      self.sessions[sid].closed(reason);
    });

    process.nextTick(function() {
      utils.invokeCallback(cb);
    });
  } else {
    process.nextTick(function() {
      utils.invokeCallback(cb);
    });
  }
};
```

踢掉用户。

```javascript
SessionService.prototype.kickBySessionId = function(sid, cb) {
  var session = this.get(sid);

  if(session) {
    // notify client
    session.closed('kick');
    process.nextTick(function() {
      utils.invokeCallback(cb);
    });
  } else {
    process.nextTick(function() {
      utils.invokeCallback(cb);
    });
  }
};
```

踢掉一个session连接。

```javascript
 SessionService.prototype.getClientAddressBySessionId = function(sid) {
   var session = this.get(sid);
   if(session) {
      var socket = session.__socket__;
      return socket.remoteAddress;
   } else {
      return null;
   }
 };
```

获得session对应客户端的远程地址。

```javascript
SessionService.prototype.sendMessage = function(sid, msg) {
  var session = this.sessions[sid];

  if(!session) {
    logger.debug('Fail to send message for non-existing session, sid: ' + sid + ' msg: ' + msg);
    return false;
  }

  return send(this, session, msg);
};
```

发送消息。

```javascript
SessionService.prototype.sendMessageByUid = function(uid, msg) {
  var sessions = this.uidMap[uid];

  if(!sessions) {
    logger.debug('fail to send message by uid for non-existing session. uid: %j',
        uid);
    return false;
  }

  for(var i=0, l=sessions.length; i<l; i++) {
    send(this, sessions[i], msg);
  }
};
```

向用户对应的所有session发送消息。


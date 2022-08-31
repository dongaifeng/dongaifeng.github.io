---
title: node模块events
urlname: tsugwg
date: '2020-09-08 18:59:14 +0800'
tags: []
categories: []
---

所有能触发事件的对象都是  `EventEmitter`  类的实例。

`1 eventEmitter.on()`  用于注册监听事件， `eventEmitter.emit()`  用于触发事件

```javascript
const EventEmitter = require("events");
class MyEmitter extends EventEmitter {}
const myEmitter = new MyEmitter();
myEmitter.on("event", () => {
  console.log("触发事件");
});
myEmitter.emit("event");
```

2 `eventEmitter.emit()`  可以传任意数量的参数到监听器函数。 当监听器函数被调用时， `this`  关键词会被指向监听器所绑定的  `EventEmitter`  实例。

```javascript
myEmitter.emit('event', 'a', 'b');

myEmitter.on('event', function(a, b) {
  console.log(a, b, this, this === myEmitter);
});
// 输出：
a
b
MyEmitter {
  domain: null,
   _events: { event: [Function] },
   _eventsCount: 1,
  _maxListeners: undefined
}
true
```

3 `EventEmitter`  会按照监听器注册的顺序同步地调用所有监听器。
可以用  `setImmediate()`  或  `process.nextTick()`  切换到异步模式。

4 `eventEmitter.once()`  只调用一次。

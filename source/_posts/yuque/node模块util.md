---
title: node模块util
urlname: uilxbi
date: '2020-09-08 18:59:14 +0800'
tags: []
categories: []
---

#### util.callbackify(fn)

将一个异步函数   或普通函数但是返回 promise  变成   中间件的形式 (err, value) =>
回调函数是异步执行   会错误跟踪

```javascript
const util = require("util");

async () => "hello word";

const callbackFunction = util.callbackify(fn);

callbackFunction((err, ret) => {
  if (err) throw err;
  console.log(ret);
});
```

#### util.inherits

将一个基础对象   继承于另一个对象

```javascript
function Base() {
  this.name = "base";
  this.base = 1991;
  this.sayHello = function () {
    console.log("Hello " + this.name);
  };
}

function Sub() {
  this.name = "Sub";
}
util.inherits(Sub, Base);
```

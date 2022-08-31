---
title: express的路由
urlname: kmb9sm
date: '2020-09-08 18:59:14 +0800'
tags: []
categories: []
---

## 一种特殊的路由

用于所有 http 请求方法的中间件。匹配“ / secret”路径的所有请求方法。

```javascript
app.all("/secret", function (req, res, next) {
  console.log("Accessing the secret section ...");
  next(); // 需要调用next() 进入下一个中间件
});
```

## 路由的路径可用正则

字符?    匹配?前面的字符 0 次或 1 次                 eg:   '/ab?cd'    匹配    ` acd，``abcd `
字符+    匹配+号前面的字符 1 次或 n 次            eg:   '/ab+cd'    匹配    `abcd`，`abbcd`，`abbbcd`
字符*    匹配*前面的字符 0 次或 n 次                 eg:  '/ab\*cd'    匹配    `abcd`，`abxcd`，`abRANDOMcd`，`ab123cd`
字符$   匹配最末的字符                              eg:   '/abcd$'    匹配    ` acd，``abcd，d `
字符^   匹配的字符必须在最前边                   eg:   '/^abcd'    匹配    ` acd，``abcd, a `

## 路由参数

```javascript
//  假如请求url为： /users/123/books/456
app.get("/users/:userId/books/:bookId", function (req, res) {
  res.send(req.params); // { "userId": "123", "bookId": "456" }
});
```

## 路由模块化

创建   路由模块  main

```javascript
var express = require("express");
var router = express.Router();
// 专门为 "/main" 设置的中间件
router.use(function timeLog(req, res, next) {
  console.log("Time: ", Date.now());
  next();
});
// 定义 "/main/" 路由
router.get("/", function (req, res) {
  res.send("Birds home page");
});
// 定义 "/main/about" 路由
router.get("/about", function (req, res) {
  res.send("main--About ");
});
module.exports = router;
```

在 app.js 页面   引入 main 模块   并使用

```
var main = require('./main')
// ...
app.use('/main', main)
```

## 错误处理

对于由路由处理程序和中间件调用的异步函数返回的错误，必须将它们传递给`next()`函数，Express 将捕获并处理它们

```javascript
app.get("/", function (req, res, next) {
  fs.readFile("/file-does-not-exist", function (err, data) {
    if (err) {
      next(err); // 通过next(err) 可以通知错误处理函数处理
    } else {
      res.send(data);
    }
  });
});
// try catch 方式
app.get("/", function (req, res, next) {
  setTimeout(function () {
    try {
      throw new Error("BROKEN");
    } catch (err) {
      next(err);
    }
  }, 100);
});
// Promise 方式
app.get("/", function (req, res, next) {
  Promise.resolve()
    .then(function () {
      throw new Error("BROKEN");
    })
    .catch(next); // 错误将会被错误处理函数接收
});
```

如果将任何内容传递给`next()`函数（字符串`'route'`除外），则 Express 将当前请求视为错误，并将跳过任何剩余的非错误处理路由和中间件函数。

如果序列中的回调不提供数据，只提供错误，则可以按如下方式简化此代码：

```javascript
app.get("/", [
  function (req, res, next) {
    fs.writeFile("/inaccessible-path", "data", next);
  },
  function (req, res) {
    res.send("OK");
  },
]);
```

使用 async/await  封装错误处理中间件

```javascript
//封装中间件 在Promise.resolve里处理业务逻辑 如果错误走catch
const asyncMiddleware = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch((err) => {
    next(err);
  });
};
// the async route handler
app.get(
  "/users/:id",
  asyncMiddleware(async (req, res) => {
    const userId = req.params.id;
    if (!userId) {
      throw new Error("missing id");
    }
    const user = await Users.get(userId);
    res.json(user);
  })
);
```

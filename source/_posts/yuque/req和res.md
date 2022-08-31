---
title: req和res
urlname: cm5xhs
date: '2020-09-08 18:59:14 +0800'
tags: []
categories: []
---

## req  代表 HTTP 请求

###

req.app
用 require()  方式引入模块   可在模块中通过 req.app  访问  express 实例

```javascript
//index.js
app.get("/viewdirectory", require("./mymiddleware.js"));
```

```javascript
//mymiddleware.js
module.exports = function (req, res) {
  res.send('The views directory is ' + req.app.get('views'));
});
```

### req.baseUrl  

安装路由器实例的 URL 路径   也就是   他的上层路径

### req.query

路由中每个查询字符串参数的属性  get 方式请求 ?后面的

## res  代表 HTTP 响应

## res.locals

包含响应局部变量的对象，该局部变量的范围仅限于该请求，因此仅可用于在该请求/响应周期（如果有）中。
app.locals  属性的值在应用程序的整个生命周期中保持不变，而 res.locals 属性仅在请求的生命周期内有效。

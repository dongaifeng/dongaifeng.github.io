---
title: axios源码分析
urlname: ytc5v6
date: '2021-06-01 17:00:16 +0800'
tags: []
categories: []
---

### axios

源码中导出的 axios 是通过 createInstance 创建的实例。使用的默认 options
axios.create 方法也是调用的 createInstance 函数，使用的默认 options 合并用户 options。
axios.Axios 等于 Axios 类。
createInstance 函数里面是 new Axios。并合并了 Axios.prototype。

### 构造函数 Axios

Axios 函数有 interceptors 对象包含 request 数组，response 数组。用来收集用户定义的拦截器。
Axios.prototype.request 方法中组装一个 chain 数组，形态大概是这样：

```javascript
var chain = [
  "请求成功拦截2",
  "请求失败拦截2",
  "请求成功拦截1",
  "请求失败拦截1",
  dispatch,
  undefined,
  "响应成功拦截1",
  "响应失败拦截1",
  "响应成功拦截2",
  "响应失败拦截2",
];
```

然后遍历这个 chain，形成一个长的 promise，形态大概是这样：

```javascript
var promise = Promise.resolve(config);
promise
  .then("请求成功拦截2", "请求失败拦截2")
  .then("请求成功拦截1", "请求失败拦截1")
  .then(dispatchRequest, undefined)
  .then("响应成功拦截1", "响应失败拦截1")
  .then("响应成功拦截2", "响应失败拦截2")

  .then("用户写的业务处理函数")
  .catch("用户写的报错业务处理函数");
```

所以 request 发生的时候执行长的 promise，会先执行请求拦截 -----> 请求 ------>响应拦截。

### dispatchRequest

dispatchRequest 是真正发送请求的函数。它内部的流程如下：

1. 对请求头和请求数据做一些处理，组装成 config
1. 返回适配器 adapter.then(resolve,reject)

adapter 是适配器，里面会判断如果是浏览器就使用 XMLHttpRequest。如果是 node 就使用 http。在 xhr 文件中对 XHRHttpRequest 做了封装。比如对请求头字段 Content-Type, auth, 的处理，还有一些事件如：timeout，onerror，onreadystatechange。

本文只是对源码流程做了基本的阐述，更详细的源码分析请参考：[https://mp.weixin.qq.com/s/GNYpgHo97xml0NxT93dHxQ](https://mp.weixin.qq.com/s/GNYpgHo97xml0NxT93dHxQ)

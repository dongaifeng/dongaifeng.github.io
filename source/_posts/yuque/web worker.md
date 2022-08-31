---
title: web worker
urlname: gghroo
date: '2020-09-08 19:00:22 +0800'
tags: []
categories: []
---

### 介绍

JavaScript 语言采用的是单线程模型，Web Worker 的作用，就是为 JavaScript 创造多线程环境，允许主线程创建 Worker 线程，将一些任务分配给后者运行。等到 Worker 线程完成计算任务，再把结果返回给主线程。
Worker 线程一旦新建成功，就会始终运行，不会被主线程上的活动（比如用户点击按钮、提交表单）打断。Worker 比较耗费资源，不应该过度使用，而且一旦使用完毕，就应该关闭。

### 特性

- 分配给 Worker 线程运行的脚本文件，必须与主线程的脚本文件同源。
- 无法读取 DOM 对象，无法使用 document、window、parent 这些对象。可以 navigator 对象和 location 对象。
- Worker 线程不能执行 alert()方法和 confirm()方法，但可以使用 XMLHttpRequest 对象发出 AJAX 请求。
- Worker 线程无法读取本地文件，即不能打开本机的文件系统（file://），它所加载的脚本，必须来自网络。

### 用法

#### 主线程创建

```javascript
var worker = new Worker("work.js");
worker.postMessage("Hello World");
worker.onmessage = function (event) {
  console.log(event.data);
};
worker.terminate(); // 关闭worker

worker.onerror(function (event) {});
```

#### worker 端

self 代表子线程自身，即子线程的全局对象。也可 this，也可什么都不写。

```javascript
self.addEventListener(
  "message",
  function (e) {
    console.log(event.data);
  },
  false
);
postMessage("You said: " + e.data);
self.close();
```

#### worker 引入插件

```javascript
importScripts("script1.js", "script2.js");
```

#### 主线程可以快速把数据交给 Worker。

```javascript
// Transferable Objects 格式
worker.postMessage(arrayBuffer, [arrayBuffer]);
// 例子
var ab = new ArrayBuffer(1);
worker.postMessage(ab, [ab]);
```

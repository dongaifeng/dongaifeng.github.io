---
title: XHR和http
urlname: nx2s48
date: '2021-05-28 10:12:11 +0800'
tags: []
categories: []
---

## xmlhttprequest 2.0 总结

xmlHttRequest 是浏览器的一个接口 用于与后台进行 http 交互。08 年 XMLHttpRequest2.0 问世。在 1.0 的基础上新增了一些方法。
现在浏览器基本都支持 XMLHttpRequest2.0.
1 http 请求时限

```javascript
xhr.timeout = 3000;
xhr.ontimeout = function (event) {
  alert("请求超时！");
};
```

![image.gif](https://cdn.nlark.com/yuque/0/2019/gif/462392/1572311565643-c4a295c3-d85f-4d9d-b089-04b9a15ddf6e.gif#align=left&display=inline&height=1&margin=%5Bobject%20Object%5D&name=image.gif&originHeight=1&originWidth=1&size=43&status=done&style=none&width=1)
2 FormData 对象。

```javascript
var form = document.getElementById("myform");
var formData = new FormData(form);
formData.append("secret", "123456"); // 添加一个表单项
xhr.open("POST", form.action);
xhr.send(formData);
```

![image.gif](https://cdn.nlark.com/yuque/0/2019/gif/462392/1572311565647-846a685b-756e-4c7b-b74e-c6a5795b98c6.gif#align=left&display=inline&height=1&margin=%5Bobject%20Object%5D&name=image.gif&originHeight=1&originWidth=1&size=43&status=done&style=none&width=1)
3 上传文件
 FormData 对象也可以处理   <input type="file">  浏览器就会在调用  `send()`  时构建  `multipart/form-data`  请求

```javascript
var files = document.querySelector('input[type="file"]');
for (var i = 0; i < files.length; i++) {
  formData.append(files[i].name, files[i]);
}
xhr.send(formData);
```

![image.gif](https://cdn.nlark.com/yuque/0/2019/gif/462392/1572311565648-3120c00c-2d68-4e82-8087-626cd6cfac9f.gif#align=left&display=inline&height=1&margin=%5Bobject%20Object%5D&name=image.gif&originHeight=1&originWidth=1&size=43&status=done&style=none&width=1)
4 CORS 跨域资源共享
服务器端已启用了 CORS

```javascript
Access-Control-Allow-Origin: http://example.com
Access-Control-Allow-Origin: *
```

![image.gif](https://cdn.nlark.com/yuque/0/2019/gif/462392/1572311565654-a3b6b715-34ef-4160-a61c-30a807c8a04f.gif#align=left&display=inline&height=1&margin=%5Bobject%20Object%5D&name=image.gif&originHeight=1&originWidth=1&size=43&status=done&style=none&width=1)
5  接受二进制数据

```javascript
var xhr = new XMLHttpRequest();
xhr.open("GET", "/path/to/image.png");
xhr.responseType = "blob";
// 请求回来数据这样接收
var blob = new Blob([xhr.response], { type: "image/png" });
```

![image.gif](https://cdn.nlark.com/yuque/0/2019/gif/462392/1572311565659-951a37f8-d5af-4376-9782-5e35cb94ac90.gif#align=left&display=inline&height=1&margin=%5Bobject%20Object%5D&name=image.gif&originHeight=1&originWidth=1&size=43&status=done&style=none&width=1)
xhr.responseType
在发送请求前，根据您的数据需要，将  `xhr.responseType`  设置为“text”、“arraybuffer”、“blob”或“document”。请注意，设置（或忽略）`xhr.responseType = ''`  会默认将响应设为“text”。
6 进度监测

```javascript
xhr.onprogress = updateProgress; // 这是下载的进度事件回调
xhr.upload.onprogress = updateProgress; // 这是上传进度事件回调
// event.total是需要传输的总字节，event.loaded是已经传输的字节。如果event.lengthComputable不为真，则event.total等于0
function updateProgress(event) {
  if (event.lengthComputable) {
    var percentComplete = event.loaded / event.total;
  }
}
```

![image.gif](https://cdn.nlark.com/yuque/0/2019/gif/462392/1572311565659-3c019ce1-12fe-4147-8fb1-ee35ff39edf1.gif#align=left&display=inline&height=1&margin=%5Bobject%20Object%5D&name=image.gif&originHeight=1&originWidth=1&size=43&status=done&style=none&width=1)

一个完整的请求

```javascript
let xhr = new XMLHttpRequest();
// 请求成功回调函数
xhr.onload = (e) => {
  console.log("request success");
};
// 请求结束
xhr.onloadend = (e) => {
  console.log("request loadend");
};
// 请求出错
xhr.onerror = (e) => {
  console.log("request error");
};
// 请求超时
xhr.ontimeout = (e) => {
  console.log("request timeout");
};
xhr.timeout = 0; // 设置超时时间,0表示永不超时
// 初始化请求
xhr.open("GET/POST/DELETE/...", "/url", true || false);
// 设置期望的返回数据类型 'json' 'text' 'document' ...
xhr.responseType = "";
// 设置请求头
xhr.setRequestHeader("", "");
// 发送请求
xhr.send(null || new FormData() || "a=1&b=2" || "json字符串");
```

![image.gif](https://cdn.nlark.com/yuque/0/2019/gif/462392/1572311565656-d4182b51-8ca8-4f2b-ab95-ce76aaf51914.gif#align=left&display=inline&height=1&margin=%5Bobject%20Object%5D&name=image.gif&originHeight=1&originWidth=1&size=43&status=done&style=none&width=1)

## http 详解

[超文本传输协议](https://baike.baidu.com/item/%E8%B6%85%E6%96%87%E6%9C%AC%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AE/8535513)（HTTP，HyperText Transfer Protocol)是[互联网](https://baike.baidu.com/item/%E4%BA%92%E8%81%94%E7%BD%91)上应用最为广泛的一种[网络协议](https://baike.baidu.com/item/%E7%BD%91%E7%BB%9C%E5%8D%8F%E8%AE%AE/328636)。
基于 TCP 协议。
HTTP 协议是无状态协议。无状态是指协议对于事务处理没有记忆能力。
同一网站同时只能建立 6 个 TCP 链接。

#### HTTPS 与 HTTP 的一些区别

- HTTPS 协议需要到 CA 申请证书，一般免费证书很少，需要交费。

- HTTP 协议运行在 TCP 之上，所有传输的内容都是明文，HTTPS 运行在 SSL/TLS 之上，SSL/TLS 运行在 TCP 之上，所有传输的内容都经过加密的。

- HTTP 和 HTTPS 使用的是完全不同的连接方式，用的端口也不一样，前者是 80，后者是 443。

- HTTPS 可以有效的防止运营商劫持，解决了防劫持的一个大问题。

[参考文章](https://blog.csdn.net/weixin_38087538/article/details/82838762)
[参考文章](https://mp.weixin.qq.com/mp/homepage?__biz=MzI2NTQ5NTE4OA==&hid=1&sn=d38e63d4e863c39b3db1494e81e40c97&scene=1&devicetype=Windows+10+x64&version=63010043⟨=zh_CN&nettype=WIFI&ascene=1&wx_header=1&uin=&key=&fontgear=1)

#### http2

##### 二进制分帧

htttp / 1.x 是采用的明文协议，所有的内容，人类可以阅读

```http
GET / HTTP/1.1
Host: jiajunhuang.com

```

HTTP/2 采用二进制协议，二进制协议在解析的时候更加高效。
将消息分成多个帧，每个帧有唯一 id 标识。

##### 多路复用

HTTP/2 复用 TCP 连接，在一个连接里，客户端和浏览器都可以同时发送多个请求或回应，而且不用按照顺序一一对应，这样就避免了"队头堵塞"。

##### 服务器推送

服务端可以在发送页面 HTML 时主动推送其它资源，而不用等到浏览器解析到相应位置，发起请求再响应。例如服务端可以主动把 JS 和 CSS 文件推送给客户端，而不需要客户端解析 HTML 时再发送这些请求。
服务端可以主动推送，客户端也有权利选择是否接收。如果服务端推送的资源已经被浏览器缓存过，浏览器可以通过发送 RST_STREAM 帧来拒收。主动推送也遵守同源策略，服务器不会随便推送第三方资源给客户端。

##### 头部压缩

HTTP 1.1 请求的大小变得越来越大，有时甚至会大于 TCP 窗口的初始大小，因为它们需要等待带着 ACK 的响应回来以后才能继续被发送。HTTP/2 对消息头采用 HPACK（专为 http/2 头部设计的压缩格式）进行压缩传输，能够节省消息头占用的网络的流量。而 HTTP/1.x 每次请求，都会携带大量冗余头信息，浪费了很多带宽资源。

参考：[https://zhuanlan.zhihu.com/p/26559480](https://zhuanlan.zhihu.com/p/26559480)

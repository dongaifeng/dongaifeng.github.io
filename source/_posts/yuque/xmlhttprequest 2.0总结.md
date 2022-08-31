---
title: xmlhttprequest 2.0总结
urlname: bbxpfo
date: '2020-09-08 19:00:22 +0800'
tags: []
categories: []
---

xmlHttRequest 是浏览器的一个接口 用于与后台进行 http 交互。08 年 XMLHttpRequest2.0 问世。在 1.0 的基础上新增了一些方法。
现在浏览器基本都支持 XMLHttpRequest2.0.
1 http 请求时限

```javascript
xhr.timeout = 3000;
xhr.ontimeout = function (event) {
  alert("请求超时！");
};
```

![image.gif](https://cdn.nlark.com/yuque/0/2019/gif/462392/1572311565643-c4a295c3-d85f-4d9d-b089-04b9a15ddf6e.gif#align=left&display=inline&height=1&name=image.gif&originHeight=1&originWidth=1&size=43&status=done&width=1)
2 FormData 对象。

```javascript
var form = document.getElementById("myform");
var formData = new FormData(form);
formData.append("secret", "123456"); // 添加一个表单项
xhr.open("POST", form.action);
xhr.send(formData);
```

![image.gif](https://cdn.nlark.com/yuque/0/2019/gif/462392/1572311565647-846a685b-756e-4c7b-b74e-c6a5795b98c6.gif#align=left&display=inline&height=1&name=image.gif&originHeight=1&originWidth=1&size=43&status=done&width=1)
3 上传文件
 FormData 对象也可以处理   <input type="file">  浏览器就会在调用  `send()`  时构建  `multipart/form-data`  请求

```javascript
var files = document.querySelector('input[type="file"]');
for (var i = 0; i < files.length; i++) {
  formData.append(files[i].name, files[i]);
}
xhr.send(formData);
```

![image.gif](https://cdn.nlark.com/yuque/0/2019/gif/462392/1572311565648-3120c00c-2d68-4e82-8087-626cd6cfac9f.gif#align=left&display=inline&height=1&name=image.gif&originHeight=1&originWidth=1&size=43&status=done&width=1)
4 CORS 跨域资源共享
服务器端已启用了 CORS

```javascript
Access-Control-Allow-Origin: http://example.com
Access-Control-Allow-Origin: *
```

![image.gif](https://cdn.nlark.com/yuque/0/2019/gif/462392/1572311565654-a3b6b715-34ef-4160-a61c-30a807c8a04f.gif#align=left&display=inline&height=1&name=image.gif&originHeight=1&originWidth=1&size=43&status=done&width=1)
5  接受二进制数据

```javascript
var xhr = new XMLHttpRequest();
xhr.open("GET", "/path/to/image.png");
xhr.responseType = "blob";
// 请求回来数据这样接收
var blob = new Blob([xhr.response], { type: "image/png" });
```

![image.gif](https://cdn.nlark.com/yuque/0/2019/gif/462392/1572311565659-951a37f8-d5af-4376-9782-5e35cb94ac90.gif#align=left&display=inline&height=1&name=image.gif&originHeight=1&originWidth=1&size=43&status=done&width=1)
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

![image.gif](https://cdn.nlark.com/yuque/0/2019/gif/462392/1572311565659-3c019ce1-12fe-4147-8fb1-ee35ff39edf1.gif#align=left&display=inline&height=1&name=image.gif&originHeight=1&originWidth=1&size=43&status=done&width=1)

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

![image.gif](https://cdn.nlark.com/yuque/0/2019/gif/462392/1572311565656-d4182b51-8ca8-4f2b-ab95-ce76aaf51914.gif#align=left&display=inline&height=1&name=image.gif&originHeight=1&originWidth=1&size=43&status=done&width=1)

---
title: node实现上传文件
urlname: raydab
date: '2020-09-08 18:59:14 +0800'
tags: []
categories: []
---

```html
<html>
  <head>
    <title>file test</title>
    <script>
      window.onload = function () {
        var files = document.getElementsByTagName("input"),
          len = files.length,
          file;
        for (var i = 0; i < len; i++) {
          file = files[i];
          if (file.type !== "file") continue; // 不是文件类型的控件跳过
          // 监听文件改变，向后台发送请求（xhr 方式），设置请求头为上传文件的name
          file.onchange = function () {
            var _files = this.files;
            if (!_files.length) return;
            if (_files.length === 1) {
              // 选择单个文件
              var xhr = new XMLHttpRequest();
              xhr.open("POST", "http://localhost:3000/upload");
              var filePath = files[0].value;
              // 设置请求头加入 文件名称项 后台可以获取请求头里的name
              xhr.setRequestHeader(
                "file-name",
                filePath.substring(filePath.lastIndexOf("\\") + 1)
              );
              xhr.send(_files[0]);
            } else {
            }
          };
        }
      };
    </script>
  </head>
  <body>
    <input id="file1" type="file" />
  </body>
</html>
```

```javascript
const http = require("http");
const fs = require("fs");
const path = require("path");
const chunk = []; // 用于存放Buffe格式的数据
let size = 0; // Buffer的长度
const server = http.createServer((request, response) => {
  const { pathname } = require("url").parse(request.url);

  // 判断是不是上传
  if (pathname === "/upload") {
    console.log("upload....");
    // 获取文件的name
    const fileName = request.headers["file-name"]
      ? request.headers["file-name"]
      : "abc.png";
    // 定义前台传来的文件 的绝对路径
    const outputFile = path.resolve(__dirname, fileName);
    // 创建一个写入流 写入干菜创建的文件目录
    const fis = fs.createWriteStream(outputFile);

    // Buffer方式接收文件
    // Buffer connect
    // request.on('data',data => {
    //     chunk.push(data)
    //     size += data.length
    //     console.log('data:',data ,size)
    // })
    // request.on('end',() => {
    //     console.log('end...')
    //     const buffer = Buffer.concat(chunk,size)
    //     size = 0
    //     fs.writeFileSync(outputFile,buffer)
    //     response.end()
    // })

    // 流事件写入
    // request.on('data', data => {
    //     console.log('data:',data)
    //     fis.write(data)
    // })
    // request.on('end', () => {
    //     fis.end()
    //     response.end()
    // })

    // 管道方式 直接对接request
    request.pipe(fis);
    response.end();
  } else {
    // 获取文件的路径 顺便设置了首页
    const filename = pathname === "/" ? "index.html" : pathname.substring(1);
    // 获取文件后缀 并判断 返回要设置的response Content_Type 格式
    var type = (function (_type) {
      switch (
        _type // 扩展名
      ) {
        case "html":
        case "htm":
          return "text/html charset=UTF-8";
        case "js":
          return "application/javascript charset=UTF-8";
        case "css":
          return "text/css charset=UTF-8";
        case "txt":
          return "text/plain charset=UTF-8";
        case "manifest":
          return "text/cache-manifest charset=UTF-8";
        default:
          return "application/octet-stream";
      }
    })(filename.substring(filename.lastIndexOf(".") + 1));
    // 异步读取文件,并将内容作为单独的数据块传回给回调函数
    // 对于确实很大的文件,使用API fs.createReadStream()更好
    fs.readFile(filename, function (err, content) {
      if (err) {
        // 如果由于某些原因无法读取文件
        response.writeHead(404, { "Content-type": "text/plain charset=UTF-8" });
        response.write(err.message);
      } else {
        // 否则读取文件成功
        response.writeHead(200, { "Content-type": type });
        response.write(content); // 把文件内容作为响应主体
      }
      response.end();
    });
  }
});
server.listen(3000);
```

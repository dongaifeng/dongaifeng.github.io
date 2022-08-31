---
title: socket.io简单使用
urlname: ncvid5
date: '2020-09-08 18:59:14 +0800'
tags: []
categories: []
---

#### 前端

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Socket.IO chat</title>
    <style>
      * {
        margin: 0;
        padding: 0;
        box-sizing: border-box;
      }
      body {
        font: 13px Helvetica, Arial;
      }
      form {
        background: #000;
        padding: 3px;
        position: fixed;
        bottom: 0;
        width: 100%;
      }
      form input {
        border: 0;
        padding: 10px;
        width: 90%;
        margin-right: 0.5%;
      }
      form button {
        width: 9%;
        background: rgb(130, 224, 255);
        border: none;
        padding: 10px;
      }
      #messages {
        list-style-type: none;
        margin: 0;
        padding: 0;
      }
      #messages li {
        padding: 5px 10px;
      }
      #messages li:nth-child(odd) {
        background: #eee;
      }
    </style>
  </head>
  <body>
    <ul id="messages"></ul>
    <form action="">
      <input id="m" autocomplete="off" /><button>Send</button>
    </form>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/socket.io/2.2.0/socket.io.js"></script>
    <script src="http://libs.baidu.com/jquery/2.1.1/jquery.min.js"></script>
    <script>
      $(function () {
        var socket = io();
        $("form").submit(function (e) {
          e.preventDefault();

          // 触发 aaa 事件
          socket.emit("aaa", $("#m").val());
          $("#m").val("");
          return false;
        });

        // 监听bbb时间
        socket.on("bbb", function (msg) {
          $("#messages").append($("<li>").text(msg));
        });
      });
    </script>
  </body>
</html>
```

#### 服务器

```javascript
var app = require("express")();
var http = require("http").Server(app);
var io = require("socket.io")(http);

app.get("/", function (req, res) {
  res.sendFile(__dirname + "/soket.html");
});

// 连接事件
io.on("connection", function (socket) {
  socket.on("aaa", function (msg) {
    // 所有人都能收到
    // io.emit('bbb', msg)

    // 广播给除了发送者外所有人
    socket.broadcast.emit("bbb", msg);
  });

  // 断开连接事件
  socket.on("disconnect", function () {
    console.log("user disconnected");
  });
});
http.listen(3000, function () {
  console.log("listening on *:3000");
});
```

---
title: node模块fs
urlname: iay9gd
date: '2020-09-08 18:59:14 +0800'
tags: []
categories: []
---

#### fs.watch

用于以函数的方式与文件系统进行交互.监控文件夹变化.

```javascript
// 引入fs模块
const fs = require("fs");
// 通过fs.watch方法可以创建一个fs.FSWatcher类的实例。
let watcher = fs.watch(
  __dirname, // 监控的文件夹，这里用了一个模块的变量，当前js文件所在的目录
  { recursive: true }, // 是否监控子文件夹，还可以设置编码，具体参考官网文档
  (eventName, fileName) => {
    // 回调函数接受两个参数：事件名字和文件名
    console.log(`事件名字：${eventName}, 文件名字： ${fileName}`);
  }
);

// 还可以单独注册事件，回调函数跟watch方法一致。还可以监听：error事件。
watcher.on("change", (eventType, fileName) => {
  console.log("事件名：%s , 文件名： %s", eventType, fileName);
});

// 设置13秒后，退出监控文件夹
setTimeout(() => {
  // 关闭监控。
  watcher.close(function (err) {
    if (err) {
      console.error(err);
    }
    console.log("关闭watch");
  });
}, 13000);

/********
以下是输出的结果：当js文件修改的时候
事件名字：change, 文件名字： 05filewatch.js
事件名：change , 文件名： 05filewatch.js
事件名字：change, 文件名字： 05filewatch.js
事件名：change , 文件名： 05filewatch.js
*/
```

### 文件操作

#### fs.readFile / fs.readFileSync

```javascript
const fs = require("fs");
const path = require("path");
// 把当前js所在的目录中的a.html文件的路径赋值给 fileName
let fileName = path.join(__dirname, "a.html");

// 读取a.html文件，按照utf8的编码方式读取。
// 回调函数第一个参数是err(这个是一个默认的约定规范，
// 大多数node的回调函数第一个参数都是异常的err，如果为空则表示没有错误)
// 第二个参数是文件的所有内容。
fs.readFile(fileName, { encoding: "utf8" }, (err, data) => {
  if (err) throw err; // 判断是否读取错误
  console.log(data); // 文件内容读取并打印到控制台。
});

//readFileSync方法是readFile的同步版本，没有回调函数，函数的返回值就是文件内容。
// 以下代码是同步读取，不使用回调函数，此方法不是用的： libuv 的线程池的线程执行，所以慎用！！
let fileContent = fs.readFileSync(fileName, { encoding: "utf8" });
console.log(fileContent);
```

#### fs.writeFile / writeFileSync

```javascript
const fs = require("fs");
const path = require("path");
// 把当前js所在的目录中的a.html文件的路径赋值给 fileName
let fileName = path.join(__dirname, "a.html");

// 写入文件
fs.writeFile(
  fileName, // 文件名
  "<h1>我要把这些内容写入fileName文件里</h1>", // 写入文件data，可以字符串或者Buffer
  (err) => {
    // 写入成功后的回调函数
    if (err) throw err;
    console.log("文件内容已经写入！");
  }
);

// 同步的方式写入文件
fs.writeFileSync(fileName, data, options);
```

### stream

stream 是处理数据流的一种抽象接口。stream 模块   构建 流接口   的   对象。
流可以分四类：

- Readable--可读的流(比如 fs.createReadStream().
- Writable--可写的流(比如 fs.createWriteStream().
- Duplex--可读写的流
- Transform---在读写过程中可以修改和变换数据的 Duplex 流。

#### fs.createReadStream

fs.createReadStream 创建可读流。读取大文件。
可读流有监听事件: close, data, end, error, readable

```javascript
const fs = require('fs');
const path = require('path');
// 把当前js所在的目录中的a.html文件的路径赋值给 fileName
let fileName = path.join(__dirname, 'a.html');

// 创建可读流
let readStream = fs.createReadStream(fileName, {
  flags: 'r',       // 设置文件只读模式打开文件
  encoding: 'utf8'  // 设置读取文件的内容的编码
  start: 0, // 通过start 和 end 设置 文件 可读取的字节数范围
  end: 99,
  autoClose: true,   // 当 'error' 或 'end' 事件时，文件描述符会被自动地关闭
});

// 打开文件流的事件。
readStream.on('open', fd => {
  console.log('文件可读流已打开', fd);
});

// 可读流打开后，会源源不断的触发此事件方法，回调函数参数就是读取的数据。
readStream.on('data', data => {
  console.log(data);
});

readStream.on('close', () => {
  console.log('文件可读流结束！');
});
```

#### fs.createWriteStream

fs.createWriteStream 创建可写流   就是可以往这个文件写入数据
事件函数：finish，error。  结束方法：end() 。

```javascript
const fs = require("fs");
const path = require("path");
let fileNameSrc = path.join(__dirname, "jdk.dmg"); // 复制的源文件
let fileNameDist = path.join(__dirname, "jdk1.dmg"); // 拷贝完的目标文件名

// 创建可读流
let rs = fs.createReadStream(fileNameSrc, { start: 0 });

// 创建可写流
let ws = fs.createWriteStream(fileNameDist, { start: 0 });

// 可读流有监听事件: close, data, end, error, readable
rs.on("data", function (chunk) {
  if (ws.write(chunk) === false) {
    // 判断数据流是否已经写入目标了
    rs.pause(); // 可暂停data事件的触发
  }
});

setTimeout(() => {
  file.resume(); // z暂停以后 想重新读取 用此方法
}, 100);

// 当可读流结束的时候，让可写流结束。
rs.on("end", function () {
  ws.end(); // 结束可写流
  console.log("文件复制成功");
});

ws.on("drain", function () {
  rs.resume(); // 继续启动读取数据流
});
```

#### 实际项目用，使用 pipe（管道）的方式大文件的复制

```javascript
const fs = require("fs");
const path = require("path");
let fileNameSrc = path.join(__dirname, "jdk.dmg"); // 复制的源文件
let fileNameDist = path.join(__dirname, "jdk1.dmg"); // 拷贝完的目标文件名
let rs = fs.createReadStream(fileNameSrc, { start: 0 });
let ws = fs.createWriteStream(fileNameDist, { start: 0 });
rs.on("end", () => {
  console.log("读取完毕");
});
ws.on("finish", () => {
  console.log("写入成功！");
});

// 可读流直接跟可写流建立管道。可读流的数据 直接通过管道流向 可写流
rs.pipe(ws);
```

#### fs.promises

fs.promises 提过另一套 API  这一套 API 用 promise 写的   你可以用 require('fs').promises 找到

```javascript
const fsp = require("fs").promises;
fsp.readFile("./clone.js").then((data) => {
  console.log(data.toString());
});
```

#### fs.exists(path, callback)

测试给定的路径是否存在。 然后调用  callback  并带上参数  `true`  或  `false.`

```javascript
fs.exists("/etc/passwd", (exists) => {
  console.log(exists ? "存在" : "不存在");
});
```

fs 的替代方案 [fs-extra](https://www.npmjs.com/package/fs-extra)

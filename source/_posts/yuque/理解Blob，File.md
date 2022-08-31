---
title: 理解Blob，File
urlname: ehfu25
date: '2021-01-29 09:47:37 +0800'
tags: []
categories: []
---

一直以来，JS 都没有比较好的可以直接处理二进制的方法。而 Blob 的存在，允许我们可以通过 JS 直接操作二进制数据。

## Blob 是什么？怎么用？能解决什么问题？

### Blob 是什么

一个 Blob 对象 就是一个包含有 **只读原始数据 **的 **类文件 **对象。Blob 对象中的数据并不一定得是 JavaScript 中的原生形式。Blob 对象可以看做是存放二进制数据的容器，此外还可以通过 Blob 设置二进制数据的 MIME 类型。
**File** 接口基于**Blob**，继承了 blob 的功能并将其扩展使其支持用户系统上的文件。

### Blob 用法

通过构造函数创建，返回一个新创建的  `Blob`  对象，其内容由参数中给定的数组串联组成。

```javascript
var blob = new Blob(dataArr:Array<any>, opt:{type:string});
```

- dataArray：数组，包含了要添加到 Blob 对象中的数据，数据可以是任意多个 ArrayBuffer，ArrayBufferView， Blob，或者 DOMString 对象。
- opt：对象，用于设置 Blob 对象的属性（如：MIME 类型）

##### 创建一个装填 DOMString 对象的 Blob 对象

![](https://cdn.nlark.com/yuque/0/2020/png/462392/1608111664296-113641b9-16d4-480b-a419-cf574f7cdd2a.png#align=left&display=inline&height=95&margin=%5Bobject%20Object%5D&originHeight=95&originWidth=313&size=0&status=done&style=none&width=313)

##### 创建一个装填 ArrayBuffer 对象的 Blob 对象

![](https://cdn.nlark.com/yuque/0/2020/png/462392/1608111706109-c5f684af-b99b-4d9f-b27c-bbea1b61ca7d.png#align=left&display=inline&height=96&margin=%5Bobject%20Object%5D&originHeight=96&originWidth=339&size=0&status=done&style=none&width=339)

##### 创建一个装填 ArrayBufferView 对象的 Blob 对象（ArrayBufferView 可基于 ArrayBuffer 创建，返回值是一个类数组。如下：创建一个 8 字节的 ArrayBuffer，在其上创建一个每个数组元素为 2 字节的“视图”）

![](https://cdn.nlark.com/yuque/0/2020/png/462392/1608111706210-0297b93d-cbe1-4805-837d-781ff3d51567.png#align=left&display=inline&height=112&margin=%5Bobject%20Object%5D&originHeight=112&originWidth=325&size=0&status=done&style=none&width=325)

##### Blob 的属性

[`Blob.size`](https://developer.mozilla.org/zh-CN/docs/Web/API/Blob/size) ：只读 `Blob` 对象中所包含数据的大小（字节）。
`[Blob.type](https://developer.mozilla.org/zh-CN/docs/Web/API/Blob/type)` ：只读一个字符串，表明该 `Blob` 对象所包含数据的 MIME 类型。如果类型未知，则该值为空字符串

##### Blob 的方法

`Blob.slice()` 此方法返回一个新的 Blob 对象，包含了原 Blob 对象中指定范围内的数据

```javascript
Blob.slice(start:number, end:number, contentType:string)
```

- start：开始索引，默认为 0
- end：截取结束索引（不包括 end）
- contentType：新 Blob 的 MIME 类型，默认为空字符串

![](https://cdn.nlark.com/yuque/0/2020/png/462392/1608112036613-4ee07b1e-8417-46df-b93f-057461af7d22.png#align=left&display=inline&height=96&margin=%5Bobject%20Object%5D&originHeight=96&originWidth=404&size=0&status=done&style=none&width=404)

[`Blob.arrayBuffer()`](https://developer.mozilla.org/zh-CN/docs/Web/API/Blob/arrayBuffer)返回一个 promise 且包含 blob 所有内容的二进制格式的 [ArrayBuffer](https://developer.mozilla.org/zh-CN/docs/Web/API/ArrayBuffer)

```javascript
blob.arrayBuffer().then((buffer) => {
  /* buffer 处理 ArrayBuffer 数据的代码……*/
  console.log(buffer);
});
```

[`Blob.text()`](https://developer.mozilla.org/zh-CN/docs/Web/API/Blob/text)返回一个 promise 且包含 blob 所有内容的 UTF-8 格式的 [USVString](https://developer.mozilla.org/zh-CN/docs/Web/API/USVString)。
[`Blob.stream()`](https://developer.mozilla.org/zh-CN/docs/Web/API/Blob/stream)返回一个能读取 blob 内容的 [ReadableStream](https://developer.mozilla.org/zh-CN/docs/Web/API/ReadableStream)。

##### 通过 canvas.toBlob()将 canvas 转化成 blob

```
var canvas = document.getElementById("canvas");
canvas.toBlob(function(blob){
    console.log(blob);
});
```

##### 使用 Blob 创建一个 URL

```javascript
var url = URL.createObjectURL(blob);
// 会产生一个类似 blob:d3958f5c-0777-0845-9dcf-2cb28783acaf 这样的URL字符串
// 你可以像使用普通 URL 那样使用它，比如用在 img.src 上。
```

##### 从 Blob 中读取数据

```javascript
// 使用 FileReader 将 Blob的内容作为类型数组读取：
var reader = new FileReader();
reader.addEventListener("loadend", function () {
  // reader.result 包含被转化为类型数组 typed array 的 blob
});
reader.readAsArrayBuffer(blob);

// 使用Response对象将Blob中的内容读取为文本
var text = await new Response(blob).text();
```

### 使用案例

#### 分片上传

File 接口基于 Blob，继承了 Blob 的功能并进行了扩展，故我们可以像使用 Blob 一样使用 File 对象。
通过 Blob.slice 方法，可以将大文件分片，轮循向后台提交各文件片段，即可实现文件的分片上传。
分片上传逻辑如下：

- 获取要上传文件的 File 对象，根据 chunk（每片大小）对文件进行分片
- 通过 post 方法轮循上传每片文件，其中 url 中拼接 querystring 用于描述当前上传的文件信息；post body 中存放本次要上传的二进制数据片段
- 接口每次返回 offset，用于执行下次上传

下面是分片上传的简单实现：

```javascript
initUpload();

//初始化上传
function initUpload() {
  var chunk = 100 * 1024; //每片大小
  var input = document.getElementById("file"); //input file
  input.onchange = function (e) {
    var file = this.files[0];
    var query = {};
    var chunks = [];
    if (!!file) {
      var start = 0;
      //文件分片
      for (var i = 0; i < Math.ceil(file.size / chunk); i++) {
        var end = start + chunk;
        chunks[i] = file.slice(start, end);
        start = end;
      }

      // 采用post方法上传文件
      // url query上拼接以下参数，用于记录上传偏移
      // post body中存放本次要上传的二进制数据
      query = {
        fileSize: file.size,
        dataSize: chunk,
        nextOffset: 0,
      };

      upload(chunks, query, successPerUpload);
    }
  };
}

// 执行上传
function upload(chunks, query, cb) {
  var queryStr = Object.getOwnPropertyNames(query)
    .map((key) => {
      return key + "=" + query[key];
    })
    .join("&");
  var xhr = new XMLHttpRequest();
  xhr.open("POST", "http://xxxx/opload?" + queryStr);
  xhr.overrideMimeType("application/octet-stream");

  //获取post body中二进制数据
  var index = Math.floor(query.nextOffset / query.dataSize);
  getFileBinary(chunks[index], function (binary) {
    if (xhr.sendAsBinary) {
      xhr.sendAsBinary(binary);
    } else {
      xhr.send(binary);
    }
  });

  xhr.onreadystatechange = function (e) {
    if (xhr.readyState === 4) {
      if (xhr.status === 200) {
        var resp = JSON.parse(xhr.responseText);
        // 接口返回nextoffset
        // resp = {
        //     isFinish:false,
        //     offset:100*1024
        // }
        if (typeof cb === "function") {
          cb.call(this, resp, chunks, query);
        }
      }
    }
  };
}

// 每片上传成功后执行
function successPerUpload(resp, chunks, query) {
  if (resp.isFinish === true) {
    alert("上传成功");
  } else {
    //未上传完毕
    query.offset = resp.offset;
    upload(chunks, query, successPerUpload);
  }
}

// 获取文件二进制数据
function getFileBinary(file, cb) {
  var reader = new FileReader();
  reader.readAsArrayBuffer(file);
  reader.onload = function (e) {
    if (typeof cb === "function") {
      cb.call(this, this.result);
    }
  };
}

// 此功能还可以更加完善，如后台需要对合并后的文件大小进行校验；或者前端加密文件，全部上传完毕后后端解密校验等
```

#### 通过 url 下载文件

window.URL 对象可以为 Blob 对象生成一个网络地址，结合 a 标签的 download 属性，可以实现点击 url 下载文件
实现如下：

```javascript
createDownload("download.txt", "download file");

function createDownload(fileName, content) {
  var blob = new Blob([content]);
  var link = document.createElement("a");
  link.innerHTML = fileName;
  link.download = fileName;
  link.href = URL.createObjectURL(blob);
  document.getElementsByTagName("body")[0].appendChild(link);
}

// 下载的txt文件内容是：download file
```

#### 通过 url 显示图片

我们知道，img 的 src 属性及 background 的 url 属性，都可以通过接收图片的网络地址或 base64 来显示图片，同样的，我们也可以把图片转化为 Blob 对象，生成 URL（URL.createObjectURL(blob)），来显示图片。
![](https://cdn.nlark.com/yuque/0/2020/png/462392/1608113263260-1c5ca3c3-9c9b-4a6f-be29-85d7afcb3479.png#align=left&display=inline&height=206&margin=%5Bobject%20Object%5D&originHeight=206&originWidth=898&size=0&status=done&style=none&width=898)

参考链接
[MDN-blob](https://developer.mozilla.org/zh-CN/docs/Web/API/Blob)
[https://www.cnblogs.com/hhhyaaon/p/5928152.html](https://www.cnblogs.com/hhhyaaon/p/5928152.html)

## 简单理解 File

File 接口提供有关文件的信息，并允许网页中的 JavaScript 访问其内容。

通常情况下， File 对象是来自用户在一个 元素上选择文件后返回的 FileList 对象,也可以是来自由拖放操作生成的 DataTransfer 对象，或者来自 HTMLCanvasElement 上的 mozGetAsFile() API。

File  对象是特殊类型的  [Blob](https://developer.mozilla.org/zh-CN/docs/Web/API/Blob)，且可以用在任意的 Blob 类型的 context 中（new 的时候传的第一个参数）。比如说， [FileReader](https://developer.mozilla.org/zh-CN/docs/Web/API/FileReader), [URL.createObjectURL()](https://developer.mozilla.org/zh-CN/docs/Web/API/URL/createObjectURL), [createImageBitmap()](https://developer.mozilla.org/zh-CN/docs/Web/API/ImageBitmapFactories/createImageBitmap), 及  [XMLHttpRequest.send()](<https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest#send()>)  都能处理  Blob  和  File。

File 继承自 Blob，所以拥有 blob 的一切方法和属性，并且有一些自己的属性。
[`File.lastModified`](https://developer.mozilla.org/zh-CN/docs/Web/API/File/lastModified) 只读 返回当前 `File` 对象所引用文件最后修改时间。
[`File.lastModifiedDate`](https://developer.mozilla.org/zh-CN/docs/Web/API/File/lastModifiedDate) 只读 返回当前 `File` 对象所引用文件最后修改时间的 `[Date](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date)` 对象。
[`File.name`](https://developer.mozilla.org/zh-CN/docs/Web/API/File/name) 只读 返回当前 `File` 对象所引用文件的名字。
[`File.size`](https://developer.mozilla.org/zh-CN/docs/Web/API/File/size) 只读 返回文件的大小。[`File.webkitRelativePath`](https://developer.mozilla.org/zh-CN/docs/Web/API/File/webkitRelativePath) 只读返回 [`File`](https://developer.mozilla.org/zh-CN/docs/Web/API/File) 相关的 path 或 URL。[`File.type`](https://developer.mozilla.org/zh-CN/docs/Web/API/File/type) 只读 返回文件的 [多用途互联网邮件扩展类型（MIME Type）](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types)

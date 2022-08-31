---
title: 理解FileReader
urlname: bwgual
date: '2021-01-29 09:47:37 +0800'
tags: []
categories: []
---

FileReader 对象 其作用是从 Blob ，File 对象中读取数据。 它使用事件来传递数据，因为从磁盘读取数据可能比较费时间。所有可以从事件回调函数获取文件数据。

File 对象继承自 Blob，其中 File 对象可以是来自用户在一个[<input>](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/input)元素上选择文件后返回的[FileList](https://developer.mozilla.org/zh-CN/docs/Web/API/FileList)对象,也可以来自拖放操作生成的 [DataTransfer](https://developer.mozilla.org/zh-CN/docs/Web/API/DataTransfer)对象,还可以是来自在一个[HTMLCanvasElement](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLCanvasElement)上执行 mozGetAsFile()方法后返回结果。
Blob，File 对象只是二进制数据的容器，本身并不能操作二进制，所以用 FileReader 操作 Blob，File。

## 创建实例

```javascript
var reader = new FileReader();
```

## 属性

| 属性名               | 描述                                                                                                                   |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| FileReader.error     | 只读，一个[DOMException](https://developer.mozilla.org/zh-CN/docs/Web/API/DOMException)表示在读取文件时发生的 c 错误   |
| FileReader.readState | 表示状态的数字。0 未加载，1 加载中，2 完场读取                                                                         |
| FileReader.result    | 文件的内容。该属性仅在读取操作完成后才有效                                                                             |

## 方法

| 方法定义                       | 描述                                                |
| ------------------------------ | --------------------------------------------------- |
| abort():void                   | 终止文件读取操作                                    |
| readAsArrayBuffer(file):void   | 异步按字节读取文件内容，结果用 ArrayBuffer 对象表示 |
| readAsBinaryString(file):void  | 异步按字节读取文件内容，结果为文件的二进制串        |
| readAsDataURL(file):void       | 异步读取文件内容，结果用 data:url 的字符串形式表示  |
| readAsText(file,encoding):void | 异步按字符读取文件内容，结果用字符串形式表示        |

## 事件

| 事件名称    | 描述                                    |
| ----------- | --------------------------------------- |
| onabort     | 当读取操作被中止时调用                  |
| onerror     | 当读取操作发生错误时调用                |
| onload      | 当读取操作成功完成时调用                |
| onloadend   | 当读取操作完成时调用,不管是成功还是失败 |
| onloadstart | 当读取操作将要开始之前调用              |
| onprogress  | 在读取数据过程中周期性调用              |

## 使用方法

FileReader 通过异步的方式读取文件内容，结果均是通过事件回调获取，下面是一个读取本地 txt 文件内容的例子：

```javascript
var input = document.getElementById("file"); //input file
input.onchange = function () {
  var file = this.files[0];
  if (!!file) {
    //读取本地文件，以gbk编码方式输出
    var reader = new FileReader();
    reader.readAsText(file, "gbk");
    reader.onload = function () {
      //读取完毕后输出结果
      console.log(this.result);
    };
  }
};
```

此外我们还可以通过注册 onprogress、onerror 等事件，记录文件读取进度或异常行为等等。
FileReader 提供了四种不同的读取文件的方式，如：

- readAsArrayBuffer 会将文件内容读取为 ArrayBuffer 对象，
- readAsBinaryString 则将文件读取为二进制串。
- readAsDataURL 会将文件读取成 URL 格式的字符串（base64 编码）
- readAsText 读取成字符串。

## 实操

首先准备一张图片（6764 字节）和一个 txt 文本（51 字节）作为测试文件：
![](https://cdn.nlark.com/yuque/0/2020/png/462392/1608174414422-81c1db64-93dc-4a1f-a1eb-e39e61dc4a1e.png#align=left&display=inline&height=47&margin=%5Bobject%20Object%5D&originHeight=47&originWidth=558&size=0&status=done&style=none&width=558)
接着编写测试代码：

```javascript
var reader = new FileReader();
// 通过四种方式读取文件
//reader.readAsXXX(file);
reader.onload = function () {
  //查看文件输出内容
  console.log(this.result);
  //查看文件内容字节大小
  console.log(new Blob([this.result]));
};
```

### readAsDataURL

查看图片输出结果：
![](https://cdn.nlark.com/yuque/0/2020/png/462392/1608174414443-6418934c-fd13-4174-9d44-0f89bb69c947.png#align=left&display=inline&height=35&margin=%5Bobject%20Object%5D&originHeight=48&originWidth=1014&size=0&status=done&style=none&width=746)
查看 txt 输出结果：
![](https://cdn.nlark.com/yuque/0/2020/png/462392/1608174414430-aff1c9ad-8f7e-4c4b-9129-ff9ad6ad8b75.png#align=left&display=inline&height=47&margin=%5Bobject%20Object%5D&originHeight=47&originWidth=559&size=0&status=done&style=none&width=559)
很明显，readAsDataURL 会将文件内容进行 base64 编码后输出，这个很好区分。

### readAsText

此方法可以通过不同的编码方式读取字符，我们使用`utf-8`读取
查看图片输出结果：
![](https://cdn.nlark.com/yuque/0/2020/png/462392/1608174414393-f665b889-e3db-467d-bf1a-22017621d522.png#align=left&display=inline&height=234&margin=%5Bobject%20Object%5D&originHeight=234&originWidth=542&size=0&status=done&style=none&width=542)
查看 txt 输出结果：
![](https://cdn.nlark.com/yuque/0/2020/png/462392/1608174414417-9589e806-86c5-4d88-bd85-76f6892a9b36.png#align=left&display=inline&height=61&margin=%5Bobject%20Object%5D&originHeight=61&originWidth=271&size=0&status=done&style=none&width=271)
readAsText 读取文件的单位是字符，故对于文本文件，只要按规定的编码方式读取即可；
而对于媒体文件（图片、音频、视频），其内部组成并不是按字符排列，故采用 readAsText 读取，会产生乱码，同时也不是最理想的读取文件的方式

### readAsBinaryString

查看图片输出结果：
![](https://cdn.nlark.com/yuque/0/2020/png/462392/1608174414459-9db004e4-fc6a-4e70-82d7-868a147669b0.png#align=left&display=inline&height=239&margin=%5Bobject%20Object%5D&originHeight=239&originWidth=775&size=0&status=done&style=none&width=775)
查看 txt 输出结果：
![](https://cdn.nlark.com/yuque/0/2020/png/462392/1608174414415-3acb0db2-d6e2-415c-8849-c70810fd51f0.png#align=left&display=inline&height=61&margin=%5Bobject%20Object%5D&originHeight=61&originWidth=246&size=0&status=done&style=none&width=246)
与 readAsText 不同的是，readAsBinaryString 函数会按字节读取文件内容。
然而诸如 0101 的二进制数据只能被机器识别，若想对外可见，还是需要进行一次编码，而 readAsBinaryString 的结果就是读取二进制并编码后的内容。
尽管 readAsBinaryString 方法可以按字节读取文件，但由于读取后的内容被编码为字符，大小会受到影响，故不适合直接传输，也不推荐使用。
如：测试的图片文件原大小为 6764 字节，而通过 readAsBinaryString 读取后，内容被扩充到 10092 个字节

### readAsArrayBuffer

查看图片输出结果：
![](https://cdn.nlark.com/yuque/0/2020/png/462392/1608174414438-caadc4f3-0174-4e1c-9a3c-ee605b63b6d7.png#align=left&display=inline&height=50&margin=%5Bobject%20Object%5D&originHeight=50&originWidth=252&size=0&status=done&style=none&width=252)
查看 txt 输出结果：
![](https://cdn.nlark.com/yuque/0/2020/png/462392/1608174414415-3c594e86-0ede-4188-946f-6bc852e5b324.png#align=left&display=inline&height=52&margin=%5Bobject%20Object%5D&originHeight=52&originWidth=268&size=0&status=done&style=none&width=268)
与 readAsBinaryString 类似，readAsArrayBuffer 方法会按字节读取文件内容，并转换为 ArrayBuffer 对象。
我们可以关注下文件读取后大小，与原文件大小一致。
这也就是 readAsArrayBuffer 与 readAsBinaryString 方法的区别，readAsArrayBuffer 读取文件后，会在内存中创建一个 ArrayBuffer 对象（二进制缓冲区），将二进制数据存放在其中。通过此方式，我们可以直接在网络中传输二进制内容。
ArrayBuffer 中的内容对外是不可见的，若要查看其中的内容，就要引入另一个概念：类型化数组
我们可以尝试查看下刚刚通过 readAsArrayBuffer 方法读取的图片文件内容：
![](https://cdn.nlark.com/yuque/0/2020/png/462392/1608174414423-7f173d4c-79d7-48c8-88fc-a33c350cbc65.png#align=left&display=inline&height=344&margin=%5Bobject%20Object%5D&originHeight=344&originWidth=232&size=0&status=done&style=none&width=232)
可以看到，整个图片文件的 6764 个字节，被分别存储在长度为 6764 的数组中，而数组中每一个元素的值，为当前字节的十进制数值。

## 应用场景

#### 在线预览本地文件

我们知道，img 的 src 属性或 background 的 url 属性，可以通过被赋值为图片网络地址或 base64 的方式显示图片。
在文件上传中，我们一般会先将本地文件上传到服务器，上传成功后，由后台返回图片的网络地址再在前端显示。
通过 FileReader 的 readAsDataURL 方法，我们可以不经过后台，直接将本地图片显示在页面上。这样做可以减少前后端频繁的交互过程，减少服务器端无用的图片资源，代码如下：

```javascript
var input = document.getElementById("file"); // input file
input.onchange = function () {
  var file = this.files[0];
  if (!!file) {
    var reader = new FileReader();
    // 图片文件转换为base64
    reader.readAsDataURL(file);
    reader.onload = function () {
      // 显示图片
      document.getElementById("file_img").src = this.result;
    };
  }
};
```

运行效果如下：
![](https://cdn.nlark.com/yuque/0/2020/png/462392/1608174702852-cf830a5c-1905-4e5b-8165-bdd96c49be25.png#align=left&display=inline&height=104&margin=%5Bobject%20Object%5D&originHeight=104&originWidth=848&size=0&status=done&style=none&width=848)
对于图片上传，我们也可以先将图片转换为 base64 进行传输，此时由于传输的图片内容就是一段字符串，故上传接口可以当做普通 post 接口处理，当图片传输到后台后，可以在转换为文件实体存储。
当然，考虑到 base64 转换效率及其本身的大小，本方法还是适合于上传内容简单或所占内存较小的文件。

#### 二进制数据上传

HTML5 体系的建立引入了一大堆新的东西，基于 XHR2，我们可以直接上传或下载二进制内容，无需像以往一样通过 form 标签由后端拉取二进制内容。
简单整理下上传逻辑：
1、通过 input[type="file"]标签获取本地文件 File 对象
2、通过 FileReader 的 readAsArrayBuffer 方法将 File 对象转换为 ArrayBuffer
3、创建 xhr 对象，配置请求信息
4、通过 xhr.sendAsBinary 直接将文件的 ArrayBuffer 内容装填至 post body 后发送
代码实现如下：

```javascript
var input = document.getElementById("file"); // input file
input.onchange = function () {
  var file = this.files[0];
  if (!!file) {
    var reader = new FileReader();
    reader.readAsArrayBuffer(file);
    reader.onload = function () {
      var binary = this.result;
      upload(binary);
    };
  }
};
//文件上传
function upload(binary) {
  var xhr = new XMLHttpRequest();
  xhr.open("POST", "http://xxxx/opload");
  xhr.overrideMimeType("application/octet-stream");
  //直接发送二进制数据
  if (xhr.sendAsBinary) {
    xhr.sendAsBinary(binary);
  } else {
    xhr.send(binary);
  }

  // 监听变化
  xhr.onreadystatechange = function (e) {
    if (xhr.readyState === 4) {
      if (xhr.status === 200) {
        // 响应成功
      }
    }
  };
}
```

## 总结

File 对象继承自 Blob。除了 Blob 方法和属性外，File 对象还有 name 和 lastModified 属性，以及从文件系统读取的内部功能。我们通常从用户输入如 <input> 或拖放事件来获取 File 对象。

FileReader 对象可以从文件或 blob 中读取数据，可以读取为以下三种格式：

- 字符串（readAsText）。
- ArrayBuffer（readAsArrayBuffer）。
- data url，base-64 编码（readAsDataURL）。

使用 URL.createObjectURL(file) 创建一个短的 url，并将其赋给 <a> 或 <img>。这样，文件便可以下载文件或者将其呈现为图像，作为 canvas 等的一部分。

通过网络发送一个 File， XMLHttpRequest 或 fetch 等网络 API 本身就接受 File 对象。

##### 参考文档

[CDN](https://developer.mozilla.org/zh-CN/docs/Web/API/FileReader/readAsDataURL)
[https://zh.javascript.info/file](https://zh.javascript.info/file)

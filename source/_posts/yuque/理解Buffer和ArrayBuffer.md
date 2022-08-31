---
title: 理解Buffer和ArrayBuffer
urlname: ss99nb
date: '2020-12-17 11:47:13 +0800'
tags: []
categories: []
---

现代计算机中操作二进制数据的基本单位是字节（byte），故二进制数据一般以字节数组的形式存在于程序中。如：Java 中的 InputStream 于 OutputStream 类，允许通过指定大小的字节数组（如：`byte[] bytes = new byte[1024]`）对文件进行读写。
然而回到 JS，其设计之初也没有想过要处理二进制，故对于字节的概念并不清晰。再加上 JS 对数据类型的弱化，即使要表示字节数组，也只能通过普通数组的方式表示。
HTML5 的建立对前端而言是颠覆性的，一方面基于 XHR2， 使上传下载二进制内容成为可能；另一方面，WebGL/Canvas 等新技术需要 JS 与显卡之间进行大量的、实时的数据交换，而其数据通信的形式必须是二进制。这样，JS 操作二进制成为了必然。
在 JS 中，可以通过 ArrayBuffer 和类型化数组（Typed Array）在内存中对二进制数据进行操作。

## ArrayBuffer

ArrayBuffer 是一段连续的长度固定的字节序列，如：通过实例化 ArrayBuffer 对象在内存中创建一段二进制存储空间（或叫二进制缓冲区），

```
// 创建一段字节长度为8的内存空间
var buffer = new ArrayBuffer(8);
// 获取字节长度
console.log(buffer.byteLength); // 8
```

由于是连续的内存空间，故在其上进行的读写操作都会比普通 JS Array 快很多。
但需要说明的是：ArrayBuffer 只是存储数据的区域，无法进行读写。若想进行访问，需要借助类型化数组（Typed Array）
故可以理解为：类型化数组是访问 ArrayBuffer 中数据的接口

## 类型化数组

类型化数组（或称视图 view）是读写 ArrayBuffer 中数据的接口,JS 可以通过 8 种不同的接口创建类型化数组，分别为：

| 名称         | 描述            | 字节长度 |
| ------------ | --------------- | -------- |
| Int8Array    | 8 位有符号整数  | 1        |
| Uint8Array   | 8 位无符号整数  | 1        |
| Int16Array   | 16 位有符号整数 | 2        |
| Uint16Array  | 16 位无符号整数 | 2        |
| Int32Array   | 32 位有符号整数 | 4        |
| Uint32Array  | 32 位无符号整数 | 4        |
| Float32Array | 32 位浮点数     | 4        |
| Float64Array | 64 位浮点数     | 8        |

通过类型化数组可以对 ArrayBuffer 中的数据进行读写，一段 ArrayBuffer 上可以重叠多个类型化数组。

```
// 创建一段12字节的ArrayBuffer
var b = new ArrayBuffer(12);
// 在b上创建一个视图v1，视图中每个元素类型为Uint8（占1字节），开始于字节索引0，结束于ArrayBuffer结尾
var v1 = new Uint8Array(b);
// 在b上创建一个视图v2，视图中每个元素类型为Uint32（占4字节），开始于字节索引4，结束于ArrayBuffer结尾
var v2 = new Uint32Array(b,4);
// 在b上创建一个视图v3，视图中每个元素类型为Uint16（占2字节），开始于字节索引2，视图长度为2，结束于字节索引5
var v3 = new Uint16Array(b,2,2);
```

## Buffer

Buffer 是 Node.JS 中用于操作  ArrayBuffer  的视图，是  TypedArray（类型化数组）  的一种。
当我们创建了一个 Buffer 对象后，我们可以通过 Buffer 对象的 buffer 属性来直接访问其对应的 ArrayBuffer 对象。
从 Node 的代码来看，一个 Buffer 对象（或者说是 FastBuffer）继承自 Uint8Array。
而 Uint8Array 则是 8 位无符号整型数组（一段以 8bit 数据为单位的无符号整型数组），是 ArrayBuffer 的一种。
Buffer  约等于  Uint8Array  因为  Buffer  就是通过继承  Uint8Array  实现的（ Uint8Array  是 9 种  TypedArray  视图中的一种。）。
ArrayBuffer 是一块内存，直接存储二进制数据。打个比方，如果说 ArrayBuffer 像是一块标本，而这些视图像是一个显微镜操作台，你可以看(读)到这个 ArrayBuffer 但是你需要这些 TypedArray （或 DataView）视图来操作这些数据。

```javascript
// ArrayBuffer 转 Buffer
var arrayBuffer = new ArrayBuffer(16);
const buffer = Buffer.from(arrayBuffer);
console.log(buffer.buffer === arrayBuffer); // true

// Buffer 转 ArrayBuffer
const buffer = Buffer.from("this is a test");
var arrayBuf = buffer.buffer;
console.log(buffer.toString()); // 打印出：this is a test
```

[DataView](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/DataView) 视图是一个可以从 ArrayBuffer 对象中读写多种数值类型的底层接口，使用它时，不用考虑不同平台的字节序问题。
即 ArrayBuffer 为二进制数据的存储区域。但是 ArrayBuffer 的读写，都需要通过 DataView 来执行。同时 DataView 提供有.buffer 可以直接返回其操作的 ArrayBuffer

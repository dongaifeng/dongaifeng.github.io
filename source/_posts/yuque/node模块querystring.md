---
title: node模块querystring
urlname: knc1gu
date: '2020-09-08 18:59:14 +0800'
tags: []
categories: []
---

`querystring`  模块提供用于解析和格式化 URL 查询字符串的实用工具。

## querystring.parse()将 url 查询字符串   解析为对象

querystring(str, separator, eq, options)
     str  指的是要解析的 url  查询字符串
     separator  指定   分割字符串的符号   默认是&  此项可省略
     eq  指定   划分生成对象键值   的分隔符   默认是=  可以省略
     options  对象   maxKeys：number 类型   指定解析最多多少个键值对   默认 1000
decodeURIComponent： function 类型    对含有%的字符串进行解码

```typescript
querystring.parse("name=whitemu#sex=man#sex=women", "#", null, { maxKeys: 2 });
// { name: 'whitemu', sex: 'man' }
```

## querystring.stringify()将对象转化成字符串

querystring.string(obj, seqarator, eq, options)
obj  要解析的对象
seqarator  用连接每个键值对   之间的符号   默认&  可省
eq  用于键和值   之间的符号   默认=  可省
options  对象   可省   encodeURIComponent: function  可以将一个不安全的 url 字符串转换成百分比的形式，默认值为 querystring.escape()

```typescript
querystring.stringify({ name: "whitemu", sex: ["man", "women"] }, "*", "$");
// 'name$whitemu*sex$man*sex$women'
```

## querystring.escape()  将传入字符串进行编码

```typescript
querystring.escape("name=慕白");
//  'name%3D%E6%85%95%E7%99%BD'
```

## querystring.unescape()  将含有%的字符串进行解码

```typescript
querystring.unescape("name%3D%E6%85%95%E7%99%BD");
// "name=慕白"
```

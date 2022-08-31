---
title: webpack的require.context
urlname: hskg8l
date: '2020-09-08 19:00:22 +0800'
tags: []
categories: []
---

## require.context

用于获取某个文件夹下的所有文件，他会遍历指定文件夹，然后各个导入。
require.context(dir, useSub, regExp)  接受三个参数
1  要读取的文件夹
2  是否读取子文件夹
3  一个正则   匹配文件名字

```javascript
const req = require.context("./svg", false, /\.svg$/);
```

返回一个函数   并且这个函数有三个属性
1 resolve  函数   接受一个参数 request,request 为 test 文件夹下面匹配文件的相对路径,返回这个匹配文件相对于整个工程的相对路径
2 keys  也是一个函数，返回一个数组   返回匹配成功模块的名字组成的数组
3 id  上下文模块里面所包含的模块 id. 它可能在你使用 module.hot.accept 的时候被用到。

![image.png](https://cdn.nlark.com/yuque/0/2020/png/462392/1578033484119-298b82eb-3847-4d75-8b14-b93c6a12a505.png#align=left&display=inline&height=371&name=image.png&originHeight=378&originWidth=669&size=83759&status=done&style=none&width=657)

- require.context 返回的函数是 require 函数   参数是相对路径   可以直接使用。

```javascript
// req是一个函数 传入相对路径 引入模块
const req = require.context("./svg", false, /\.svg$/);
// 直接放到map里使用
req.keys().map(req);
```

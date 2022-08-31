---
title: webpack学习3
urlname: wgfu3g
date: '2021-05-28 10:12:12 +0800'
tags: []
categories: []
---

## require.context

用于获取某个文件夹下的所有文件，他会遍历指定文件夹，然后各个导入。
require.context(dir, useSub, regExp)  接受三个参数

1.  dir 要读取的文件夹
1.  useSub 是否读取子文件夹
1.  regExp 用正则匹配文件名字

```javascript
const req = require.context("./svg", false, /\.svg$/);
```

上面的例子返回一个函数   并且这个函数有三个属性

1.  resolve  函数   接受一个参数 request，request 为 svg 文件夹下面匹配文件的相对路径,返回这个匹配文件相对于整个工程的相对路径
1.  keys  也是一个函数，返回一个数组   返回匹配成功模块的名字组成的数组
1.  id  上下文模块里面所包含的模块 id. 它可能在你使用 module.hot.accept 的时候被用到。

下面图片是 require.context 的源码
![image.png](https://cdn.nlark.com/yuque/0/2020/png/462392/1578033484119-298b82eb-3847-4d75-8b14-b93c6a12a505.png#align=left&display=inline&height=371&margin=%5Bobject%20Object%5D&name=image.png&originHeight=378&originWidth=669&size=83759&status=done&style=none&width=657)

- require.context 返回的函数是 require 函数   参数是相对路径   可以直接使用。

```javascript
// req是一个函数 传入相对路径 引入模块
const req = require.context("./svg", false, /\.svg$/);
// 直接放到map里使用
req.keys().map(req);
```

项目中使用案例
![微信图片_20191012160631.png](https://cdn.nlark.com/yuque/0/2020/png/462392/1599820416798-d9409b0c-a89a-480a-82fc-f550b30e13d0.png#align=left&display=inline&height=218&margin=%5Bobject%20Object%5D&name=%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20191012160631.png&originHeight=218&originWidth=644&size=8566&status=done&style=shadow&width=644)

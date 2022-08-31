---
title: node模块PATH
urlname: zc9dwf
date: '2020-09-08 18:59:14 +0800'
tags: []
categories: []
---

path 模块用于处理文件路径和目录的工具

1 `path.basename()`  方法返回  `path`  的最后一部分



```javascript
path.basename("/foo/bar/baz/asdf/quux.html");
// 返回: 'quux.html'
```

2 `path.dirname()`  方法返回  `path`  的目录名   不包括   文件名

```java
path.dirname('/foo/bar/baz/asdf/quux');
// 返回: '/foo/bar/baz/asdf'
```

3 `path.extname()`  方法返回  `path`  的扩展名   只抓取最后一个点和后面的扩展名。

```
path.extname('index.coffee.md');
// 返回: '.md'
```

4 `path.parse()`  方法返回一个对象，对象的属性表示  `path`  的元素。返回的对象有以下属性：`root`、`dir`、`base`、`ext`、`name。`
`path.format()`  方法会从一个对象返回一个路径字符串。 与  [`path.parse()`](https://link.jianshu.com/?t=http%3A%2F%2Fnodejs.cn%2Fapi%2Fpath.html%23path_path_parse_path)  相反。

```javascript
path.parse('/home/user/dir/file.txt');
返回:
{ root: '/',
   dir: '/home/user/dir',
   base: 'file.txt',
  ext: '.txt',
  name: 'file'
}
```

5 `path.isAbsolute()`  方法检测  `path`  是否为绝对路径。

```
path.isAbsolute('/baz/..');  // true
path.isAbsolute('qux/');     // false
```

6 `path.join()`  方法把传入的参数连接起来形成 url。

```typescript
path.join("/foo", "bar", "baz/asdf", "quux", "..");
// 返回: '/foo/bar/baz/asdf'
```

7 `path.normalize()`  方法会规范化给定的  `path`，并解析  `'..'`  和  `'.'`  片段。

```
path.normalize('/foo/bar//baz/asdf/quux/..');
// 返回: '/foo/bar/baz/asdf'
```

8 `path.resolve()`  方法会把一个路径或路径片段的序列解析为一个绝对路径。

```
path.resolve('/foo/bar', './baz');
// 返回: '/foo/bar/baz'
```

9  path.sep  表示分隔符   posix 上是 /     win 上是   \
    path.delimiter    表示间隔符  posix  是： win 是；

10  路径区别
dirname 与\_\_filename 总是返回文件的绝对路径，即物理磁盘上的路径
process.cwd()总是返回执行 node 命令时所在的文件夹路径，当前在哪里启动的脚本路径
./在 require 方法中总是相对当前文件所在文件夹的路径；在其他的地方和 process.cwd()一样，相对 node 启动文件夹

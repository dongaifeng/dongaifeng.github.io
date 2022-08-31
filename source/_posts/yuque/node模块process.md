---
title: node模块process
urlname: wkvutg
date: '2020-09-08 18:59:14 +0800'
tags: []
categories: []
---

process 对象是一个全局变量，提供进程的信息并对其进行控制。 无需使用 require()。

####  process.argv  返回一个数组，包含启动 node 进程的   命令   中的参数  

- 其中第一个元素 argv[0], 是  process.execPath
- 第二个参数是执行文件的路径

```
$ node apiTest/process.js one=1 --inspect --version // 当输入这个命令时
argv返回：
0: /usr/local/bin/node
1: /Users/mobike/Documents/webProjects/testNode/apiTest/process.js
2: one=1
3: --inspect
4: --version
/usr/local/bin/node // 这是process.execPath输出的
```

#### process.env  属性返回包含用户环境的对象

```
{
  TERM: 'xterm-256color',
  SHELL: '/usr/local/bin/bash',
  USER: 'maciej',
  PATH: '~/.bin/:/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin',
  PWD: '/Users/maciej',
  EDITOR: 'vim',
  SHLVL: '1',
  HOME: '/Users/maciej',
  LOGNAME: 'maciej',
  _: '/usr/local/bin/node'
}
```

在 webpack 打包过程中常用`process.env.NODE_ENV`判断生产环境或开发环境，`process.env`是没有`NODE_ENV`这个属性的，你可以在系统环境变量中配置。也可以在项目程序直接设置`process.env.NODE_ENV=‘dev’`。

#### process.cwd() 方法返回 Node.js 进程的当前工作目录。和 linux  的 $ pwd  命令类似

#### process.nextTick(fn)将 fn 添加到下一个时间点的队列

Timers 是 node 中定时器
    setImmediate  当前回合的  node 时间循环结束后   才去调用它的回调函数
    返回 immediate  用于   [clearImmediate()](http://nodejs.cn/s/tn26EY)。

```javascript
// 下面举例说明process.nextTick(fn)与setImmediate(fn)与setTimeout(fn,0)之间的区别
setImmediate(()=>{
  console.log("setImmediate")
});
setTimeout(()=>{
  console.log("setTimeout 0")
},0);
setTimeout(()=>{
  console.log("setTimeout 100")
},100);
process.nextTick(()=>{
  console.log("nextTick")
  process.nextTick(()=>{
    console.log("nextTick inner")
  })
});

-----输出-----
nextTick
nextTick inner
setTimeout 0
setImmediate
setTimeout 100
```

process.nextTick()中的回调函数最快执行，因为它将异步事件插入到当前执行队列的末尾
但如果 process.nextTick()中的事件执行时间过长，后面的异步事件就被延迟。
setImmediate()执行最慢，因为它将事件插入到下一个事件队列的队首，不会影响当前事件队列的执行。当 setTimeout(fn, 0)是在 setImmediate()之前执行。

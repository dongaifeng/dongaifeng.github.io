---
title: nextTick
urlname: ymmkgl
date: 2020-04-24 10:26:01 +0800
tags: []
categories: []
---

#### event loop

理解 vue 的 nextTick 之前首先要理解浏览器的 event loop 事件循环机制。

- 宏任务 macrotask 也叫 tasks。比如 setTimeout，setImmediate（node 独有），UI rendering（浏览器独有）
- 微任务 microtask 也叫 jobs。比如 promise，MutationObserver。

#### 流程

1. 执行全局 script 同步代码
1. 同步代码执行完，调用栈 Stack 空了
1. 执行微任务队列里的任务。**执行过程中产生的微任务会放到队尾，在这个周期里调用**
1. 从宏任务里取出**一个**，放到执行栈 Stack 中执行。
1. 看看微任务队列里有没有任务，有就执行。
1. 微任务执行完，重复 4 的步骤。

[参考文章](https://segmentfault.com/a/1190000016278115?utm_source=tag-newest)
[参考文章](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)

#### 解读源码

```javascript
// 存放异步函数
const callbacks = [];
// 调用标志
let pending = false;
// 清空callbacks函数的方法，循环执行里面存的异步函数
function flushCallbacks() {
  pending = false;
  const copies = callbacks.slice(0);
  callbacks.length = 0;
  for (let i = 0; i < copies.length; i++) {
    copies[i]();
  }
}

// 生成timerFunc
// 如果平台支持Promise，就用Promise做异步，把函数放到微任务队列里
// 如果不是IE 而且支持MutationObserver 就用MutationObserver，他是微任务
// 如果平台支持setImmediate，就用setImmediate，他是宏任务
// 最后保底的使用setTimeout，他是宏任务

let timerFunc;

if (typeof Promise !== "undefined" && isNative(Promise)) {
  const p = Promise.resolve();
  timerFunc = () => {
    p.then(flushCallbacks);
  };
  isUsingMicroTask = true;
} else if (
  !isIE &&
  typeof MutationObserver !== "undefined" &&
  (isNative(MutationObserver) ||
    // PhantomJS and iOS 7.x
    MutationObserver.toString() === "[object MutationObserverConstructor]")
) {
  let counter = 1;
  const observer = new MutationObserver(flushCallbacks);
  const textNode = document.createTextNode(String(counter));
  observer.observe(textNode, {
    characterData: true,
  });
  timerFunc = () => {
    counter = (counter + 1) % 2;
    textNode.data = String(counter);
  };
  isUsingMicroTask = true;
} else if (typeof setImmediate !== "undefined" && isNative(setImmediate)) {
  timerFunc = () => {
    setImmediate(flushCallbacks);
  };
} else {
  timerFunc = () => {
    setTimeout(flushCallbacks, 0);
  };
}

export function nextTick(cb?: Function, ctx?: Object) {
  let _resolve;
  // 把用户传来的函数cb添加进callbacks里
  // 调用timerFunc函数，因为timerFunc是异步的，会等到所有同步任务执行完再执行
  // timerFunc里面会调用flushCallbacks
  callbacks.push(() => {
    if (cb) {
      try {
        cb.call(ctx);
      } catch (e) {
        handleError(e, ctx, "nextTick");
      }
    } else if (_resolve) {
      _resolve(ctx);
    }
  });
  if (!pending) {
    pending = true;
    timerFunc();
  }
  // $flow-disable-line
  if (!cb && typeof Promise !== "undefined") {
    return new Promise((resolve) => {
      _resolve = resolve;
    });
  }
}
```

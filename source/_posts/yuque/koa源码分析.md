---
title: koa源码分析
urlname: gahk2p
date: '2021-06-01 17:01:01 +0800'
tags: []
categories: []
---

首先对 Koa 类进行分析，Koa 继承了原生的 Emitter 类。主要方法有 use，listen。
use 是添加中间件，就是向 middleware 数组 push 函数参数。
listren 会调用原生的 http.createServer 创建服务，并且传入 handleRequest 回调函数。
在 callback 方法里调用了 koa-compose 函数将中间件数组变成一个函数，这里是 koa 的洋葱圈模型的重点。

```javascript
// koa 继承于node原生的Emitter类
class Application extends Emitter {
  constructor(options) {
    super();
    options = options || {};
    this.middleware = [];
    this.context = Object.create(context);
    this.request = Object.create(request);
    this.response = Object.create(response);
  }

  listen(...args) {
    const server = http.createServer(this.callback());
    return server.listen(...args);
  }

  // use 会把中间件加入到数组
  use(fn) {
    this.middleware.push(fn);
    return this;
  }

  callback() {
    const fn = compose(this.middleware);
    const handleRequest = (req, res) => {
      const ctx = this.createContext(req, res);
      return this.handleRequest(ctx, fn);
    };

    return handleRequest;
  }

  handleRequest(ctx, fnMiddleware) {
    const onerror = (err) => ctx.onerror(err);
    const handleResponse = () => respond(ctx);
    return fnMiddleware(ctx).then(handleResponse).catch(onerror);
  }
}
```

中间件的使用方法

```javascript
app.use(async (ctx, next) => {
  const start = Date.now();
  await next();
  const ms = Date.now() - start;
  ctx.set("X-Response-Time", `${ms}ms`);
});

app.use(async (ctx) => {
  ctx.body = "Hello World";
});
```

compose 函数实现洋葱圈模式

```javascript
function compose(middleware) {
  // 对参数是不是数组，每一项是不是函数做校验
  if (!Array.isArray(middleware))
    throw new TypeError("Middleware stack must be an array!");
  for (const fn of middleware) {
    if (typeof fn !== "function")
      throw new TypeError("Middleware must be composed of functions!");
  }

  return function (context, next) {
    // last called middleware #
    let index = -1;
    return dispatch(0);
    function dispatch(i) {
      if (i <= index)
        return Promise.reject(new Error("next() called multiple times"));
      index = i;
      let fn = middleware[i];
      if (i === middleware.length) {
        fn = next;
      }
      if (!fn) return Promise.resolve();
      try {
        return Promise.resolve(fn(context, dispatch.bind(null, i + 1)));
      } catch (err) {
        return Promise.reject(err);
      }
    }
  };
}
```

上面的逻辑比较难以理解，其中原理就是将数组转化成一个函数，在这个函数里中间件从第一个开始调用，下一个中间件会作为上一个中间的第二个参数 next 传入，在其内部调用。
比如以下函数，就是对上面代码逻辑的实现。

```javascript
const [fn1, fn2, fn3] = this.middleware;
const fnMiddleware = function (context) {
  return Promise.resolve(
    fn1(context, function next() {
      return Promise.resolve(
        fn2(context, function next() {
          return Promise.resolve(
            fn3(context, function next() {
              return Promise.resolve();
            })
          );
        })
      );
    })
  );
};

fnMiddleware(ctx).then(handleResponse).catch(onerror);
```

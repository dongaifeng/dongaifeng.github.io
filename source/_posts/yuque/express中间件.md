---
title: express中间件
urlname: gv57mh
date: '2020-09-08 18:59:14 +0800'
tags: []
categories: []
---

## 中间件

### 主要功能

1  执行任何代码   2  更改 req 和 res   3  结束请求-响应周期   4  调用堆栈中的下一个中间件。

```javascript
//要加载中间件功能，请调用app.use()
//例如，以下代码myLogger在路由到根路径（/）之前加载中间件功能
var myLogger = function (req, res, next) {
  console.log("LOGGED");
  next();
};
app.use(myLogger);

app.get("/", function (req, res) {
  res.send("Hello World!");
});
```

`next()`函数不是 Node.js 或 Express API 的一部分，  中间件函数的第三个参数， `next()`函数可以命名为任何名称，但是通常命名为 next。

中间件加载的顺序很重要,  先加载的中间件功能也将先执行
如果`myLogger`在到达根路径的路由之后加载，则永远不会被调用，不会显示“ LOGGED”，因为根路径的路由处理程序会终止请求-响应周期。

### requestTime  给  req 添加的属性   在下面的   路由，中间件   里都可以使用。

```
var requestTime = function (req, res, next) {
  req.requestTime = Date.now()
  next()
}
app.use(requestTime)
app.get('/', function (req, res) {
  var responseText = 'Hello World!<br>'
  responseText += '<small>Requested at: ' + req.requestTime + '</small>'
  res.send(responseText)
})
```

### 中间件也可以传参数

```
module.exports = function(options) {
  return function(req, res, next) {
    // Implement the middleware function based on the options object
    next()
  }
}
```

```
var mw = require('./my-middleware.js')
app.use(mw({ option1: '1', option2: '2' }))
```

## 应用层中间件

没有设置路径的中间件，  每次应用收到请求时，都会执行该功能

```
app.use(function (req, res, next) {
  console.log('Time:', Date.now())
  next()
})
```

只对匹配`/user/:id`路径的 GET 请求起作用。

```
app.use('/user/:id', function (req, res, next) {
  console.log('Request Type:', req.method)
  next()
})
```

中间件可传多个回调函数，不要忘记 next（）

```
app.use('/user/:id', function (req, res, next) {
  console.log('中间件1')
  next()
}, function (req, res, next) {
  console.log('中间件2')
  next()
})
```

这条很没用：
一个路径可以定义多个路由，第二条路由不会引起任何问题，但是它将永远不会被调用，因为第一条路由会结束请求-响应周期

```
app.get('/user/:id', function (req, res, next) {
  console.log('ID:', req.params.id)
  next()
}, function (req, res, next) {
  res.send('User Info')
})
// handler for the /user/:id path, which prints the user ID
app.get('/user/:id', function (req, res, next) {
  res.end(req.params.id)
})
```

调用`next('route')`将控制权传递给下一条路由，仅在使用`app.METHOD()`或`router.METHOD()`函数有效

```
app.get('/user/:id', function (req, res, next) {
  // 如果id是0， 将去到2号路由 不会经过下面的中间件
  if (req.params.id === '0') next('route')
  // 如果 id 不是0 将去到下面的中间件
  else next()
}, function (req, res, next) {
  res.send('regular')
})
// 这里是2号路由
app.get('/user/:id', function (req, res, next) {
  res.send('special')
})
```

## 路由器中间件

使用`router.use()`和`router.METHOD()`函数加载路由器级中间件。
路由器级中间件与应用程序级中间件的工作方式相同，只不过它绑定到的实例`express.Router()`
以下代码的   使用路由器   实现   上边代码功能。

```
var app = express()
var router = express.Router()
//// 匹配所有路径
router.use(function (req, res, next) {
  console.log('Time:', Date.now())
  next()
})

// 匹配路径是 /user/:id 并且是get方法
router.get('/user/:id', function (req, res, next) {
   // 如果id是0， 将去到2号路由 不会经过下面的中间件
  if (req.params.id === '0') next('route')
  // 如果 id 不是0 将去到下面的中间件
  else next()
}, function (req, res, next) {

  res.render('regular')
})
// 这里是2号路由
router.get('/user/:id', function (req, res, next) {
  console.log(req.params.id)
  res.render('special')
})
// 挂载路由
app.use('/', router)
```

```
var app = express()
var router = express.Router()
// predicate the router with a check and bail out when needed
router.use(function (req, res, next) {
  if (!req.headers['x-auth']) return next('router')
  next()
})
router.get('/', function (req, res) {
  res.send('hello, user!')
})
// use the router and 401 anything falling through
app.use('/admin', router, function (req, res) {
  res.sendStatus(401)
})
```

调用`next('router')`  将控制权转回路由器实例   看清楚并不是` next('route') ``这里待研究 `

```
var app = express()
var router = express.Router()

router.use(function (req, res, next) {
// 这里将退出router，并不会走1号路由以及下面所有路由 会直接走到2号中间件
  if (!req.headers['x-auth']) return next('router')
  next()
})
这里是1号路由
router.get('/', function (req, res) {
  res.send('hello, user!')
})
// 2号中间件
app.use('/admin', router, function (req, res) {
  res.sendStatus(401)
})
```

## 错误处理中间件

错误处理中间件有*四个*参数。必须提供四个参数。即使不需要使用该`next`，也必须有。
否则，该`next`对象将被解释为常规中间件，并且将无法处理错误

```
app.use(function (err, req, res, next) {
  console.error(err.stack)
  res.status(500).send('Something broke!')
})
```

## 第三方中间件

```
var cookieParser = require('cookie-parser')

app.use(cookieParser())
```

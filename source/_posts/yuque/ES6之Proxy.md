---
title: ES6之Proxy
urlname: fb9g7w
date: '2019-11-01 14:53:22 +0800'
tags: []
categories: []
---

## Proxy

Proxy 可以理解成，在目标对象之前架设一层“拦截”，也可叫做 “代理器”。
使用方法：

```javascript
let target = { name: "hello Proxy" };
let handler = {
  get: function (target, propKey) {
    console.log(target, propKey);
    return 35;
  },
};

let newObj = new Proxy(target, handler);
newObj.age; // 35
```

Proxy 接受两个参数：

- target 目标对象，就是你要代理的对象。包括原生数组，函数，甚至另一个代理。
- handler 配置对象，就是对目标对象的一些操作的拦截。

**注意：要使得 Proxy 起作用，必须针对 Proxy 实例进行操作(newObj)，而不是针对目标对象（target）进行操作**
handler 所具有的属性：

### get()

拦截某个属性的读取操作，可以接受三个参数，依次为目标对象、属性名和 proxy 实例本身（严格地说，是操作行为所针对的对象），其中最后一个参数可选。

```javascript
// 犀牛书上给的一个特别绕的例子
var pipe = function (value) {
  var funcStack = [];
  var oproxy = new Proxy(
    {},
    {
      get: function (pipeObject, fnName) {
        if (fnName === "get") {
          return funcStack.reduce(function (val, fn) {
            return fn(val);
          }, value);
        }
        funcStack.push(window[fnName]);
        return oproxy;
      },
    }
  );

  return oproxy;
};

var double = (n) => n * 2;
var pow = (n) => n * n;
var reverseInt = (n) => n.toString().split("").reverse().join("") | 0;

pipe(3).double.pow.reverseInt.get; // 63

// 这个例子表示 receiver 指向 Proxy实例
const proxy = new Proxy(
  {},
  {
    get: function (target, key, receiver) {
      return receiver;
    },
  }
);
proxy.getReceiver === proxy; // true
```

如果一个属性不可配置（configurable）且不可写（writable），则 Proxy 不能修改该属性，否则通过 Proxy 对象访问该属性会报错。

```javascript
const target = Object.defineProperties(
  {},
  {
    foo: {
      value: 123,
      writable: false,
      configurable: false,
    },
  }
);

const handler = {
  get(target, propKey) {
    return "abc";
  },
};

const proxy = new Proxy(target, handler);

proxy.foo;
// TypeError: Invariant check failed
```

### set()

拦截某个属性的赋值操作，可以接受四个参数，依次为目标对象、属性名、属性值和 Proxy 实例本身，其中最后一个参数可选。
注意：

- set 代理应当返回一个布尔值。严格模式下，set 代理如果没有返回 true，就会报错。
- 如果目标对象自身的某个属性不可写，那么 set 方法将不起作用。

```javascript
// 一个验证第四个参数的例子
const handler = {
  set: function (obj, prop, value, receiver) {
    obj[prop] = receiver;
    return true;
  },
};
const proxy = new Proxy({}, handler);
proxy.foo = "bar";
proxy.foo === proxy; // true
```

### apply()

拦截函数的调用、call 和 apply 操作。接受三个参数，分别是目标对象、目标对象的上下文对象（this）和目标对象的参数数组。

```javascript
var twice = {
  apply(target, ctx, args) {
    return Reflect.apply(...arguments) * 2;
  },
};
function sum(left, right) {
  return left + right;
}
var proxy = new Proxy(sum, twice);
proxy(1, 2); // 6
proxy.call(null, 5, 6); // 22
proxy.apply(null, [7, 8]); // 30
Reflect.apply(proxy, null, [9, 10]); // 38
// 另外，直接调用Reflect.apply方法，也会被拦截。
```

### construct()

拦截 new 命令, 接受三个参数：目标对象，构造函数的参数数组， 创建实例时，new 命令作用的构造函数（Proxy 的实例）。
目标对象必须是函数，方法返回的必须是一个对象，否则会报错。

### deleteProperty()

拦截 delete 操作，如果这个方法抛出错误或者返回 false，当前属性就无法被 delete 命令删除。

### defineProperty()

拦截了 Object.defineProperty()操作。

### 其他的属性：

- **has(target, propKey)**：拦截 propKey in proxy 的操作，返回一个布尔值。
- **ownKeys(target)**：拦截 Object.getOwnPropertyNames(proxy)、Object.getOwnPropertySymbols(proxy)、Object.keys(proxy)、for...in 循环，返回一个数组。该方法返回目标对象所有自身的属性的属性名，而 Object.keys()的返回结果仅包括目标对象自身的可遍历属性。
- **getOwnPropertyDescriptor(target, propKey)**：拦截 Object.getOwnPropertyDescriptor(proxy, propKey)，返回属性的描述对象。
- **getPrototypeOf(target)**：拦截 Object.getPrototypeOf(proxy)，返回一个对象。

### this 指向

一旦 proxy 代理 target，target.m()内部的 this 就是指向 proxy，而不是 target

```javascript
const target = {
  m: function () {
    console.log(this === proxy);
  },
};
const handler = {};

const proxy = new Proxy(target, handler);

target.m(); // false
proxy.m(); // true
```

## Math

Math 是数学函数，但又属于对象数据类型 typeof Math => ‘object’
console.dir(Math) 查看 Math 的所有函数方法。

```javascript
// 1，Math.abs() 获取绝对值
Math.abs(-12) = 12

// 2，Math.ceil() and Math.floor() 向上取整和向下取整
console.log(Math.ceil(12.03));//13
console.log(Math.ceil(12.92));//13
console.log(Math.floor(12.3));//12
console.log(Math.floor(12.9));//12

// 3，Math.round() 四舍五入
// 注意：正数时，包含5是向上取整，负数时包含5是向下取整。
Math.round(-16.3)  //16
Math.round(-16.5)  //16
Math.round(-16.51) //17

// 4，Math.random() 取[0,1)的随机小数
parseInt(Math.random()*10);  // [0, 10)
parseInt(Math.random()*10+1);// [1, 10]
Math.round(Math.random()*10);// [0, 10]
// 获取[n,m]之间的随机整数
Math.round(Math.random()*(m-n)+n)

// 5，Math.max() and Max.min() 获取一组数据中的最大值和最小值
Math.max(10,1,9,100,200,45,78); //100
Math.min(10,1,9,100,200,45,78); // 1

// 6，Math.pow() 获取一个值的多少次幂
//    Math.sqrt() 对数值开方
Math.pow(10，2) // 100
Math.sqrt(100)  // 10
```

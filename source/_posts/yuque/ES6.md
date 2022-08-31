---
title: ES6
urlname: af0h2y
date: '2021-05-28 10:12:10 +0800'
tags: []
categories: []
---

## class

```javascript
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }
  z = "z";
  bar() {}
}
// 两种方式相同
function Fn(x, y) {
  this.x = x;
  this.y = y;
  this.z = "z";
}

Fn.prototype.bar = function () {};
```

class 的 constructor 相当于 Fn 的 prototype.constructor
在 constructor 里定义属性 相当于 在 Fn 里面定义 比如 x，y
在 constructor 外定义属性 相当于 在 Fn 的 prototype 比如：bar
但是在 constructor 外定义定义变量 相当于在 constructor 定义的。 比如 z 等同于 this.z = 'z'
class 上除了自定义的 constructor 外 还有一个 toString
无论是 class  还是 Fn  new 出来的实例对象 都可以通过* proto*  找到 他的构造函数的 原型对象（prototype）
并可以修改上面的属性

```javascript
var p1 = new Point(2, 3);
var p2 = new Point(3, 2);

p1.__proto__.printName = function () {
  return "Oops";
};

p1.printName(); // "Oops"
p2.printName(); // "Oops"
```

{}.hasOwnProperty('prop') 这个对象自身上有没有 prop 这个属性 如果通过**proto**的属性 则返回 false
类不存在变量提升， 先声明 再使用
类的方法内部如果含有 this，它默认指向类的实例，如果单独拿来使用会找不到 this。

在构造方法中绑定 this 或者箭头函数

```
class Logger {
  constructor() {
    this.printName = this.printName.bind(this);
    this.getThis = () => this;
  }
}
```

加上 static 关键字，就表示该方法不会被实例对象继承，而是直接通过类来调用
静态方法包含 this 关键字，这个 this 指的是类本身，而不是实例对象

```
class Foo {
  static classMethod() {
    return 'hello';
  }
}
Foo.classMethod() // 'hello'
var foo = new Foo();
foo.classMethod() // 找不到
```

父类的静态方法，可以被子类继承，通过 extends。
静态方法也是可以从 super 对象上调用的。

```javascript
class Bar extends Foo {
  static classMethod() {
    return super.FooMethod();
  }
}
```

Class 内部只有静态方法，没有静态属性  
静态属性定义: Foo.name = 'static name'
如果通过 new 调用这个函数 new.target 指向这个函数，否则是 undefined

```javascript
function Person(name) {
  if (new.target !== undefined) {
    this.name = name;
  } else {
    throw new Error("必须使用 new 命令生成实例");
  }
}
```

super 关键字，它在下面这里表示父类的构造函数，也就是父类的 constructor，并重新定义父类 constructor 里面的属性。然后子类再继承到自己身上。

```javascript
class Father {
  constructor(x) {
    this.x = x;
    this.bbb = function () {};
  }
  fn() {
    console.log(this); //  谁调用 就指向谁
  }
}

class Son extends Father {
  constructor(x, y) {
    super(x); // 直接去调Father的constructor
    this.x = "son prop"; // 优先使用自己的属性
  }
}
```

子类必须在 constructor 方法中调用 super 方法，否则新建实例时会报错。这是因为子类自己的 this 对象，必须先通过父类的构造函数完成塑造，得到与父类同样的实例属性和方法，然后再对其进行加工，加上子类自己的实例属性和方法。如果不调用 super 方法，子类就得不到 this 对象。

在子类的构造函数中，只有调用 super 之后，才可以使用 this 关键字，否则会报错。这是因为子类实例的构建，基于父类实例

```javascript
class ColorPoint extends Point {
  constructor(x, y, color) {
    this.color = color; // ReferenceError
    super(x, y);
    this.color = color; // 正确
  }
}
```

{} instanceof Fn  对象是不是 Fn 构造出来的
Object.getPrototypeOf ()    从子类上获取父类    Object.getPrototypeOf(Son) === Father  
子类里调用 super()  相当于 Father.prototype.constructor.call(this)
super 也可做为对象用。也可在子类 constructor 外使用。  super 指向父类的原型对象 super = Father.prototype, 但是不能调用定义在父类 constructor 里的属性

在子类静态方法之中，super 将指向父类(Father)，而不是父类的原型对象, super.fn 会去找 Father 的静态方法。

```javascript
static myMethod(msg) {  // 子类的静态方法
    super.myMethod(msg); // 会去找父类 static 的方法
  }
```

子类的**proto**总是指向父类。
子类 prototype.**proto**，表示方法的继承，总是指向父类的 prototype 属性。

## Map，Set

#### Set

Set 是一种数据结构。类似数组，但是成员的值都唯一。Set 是一个能够存储 **无重复值的有序列表** 的容器。

```javascript
const set = new Set([1, "a", 3, 4, 4]);
set.add(5);
set.delete(3);
console.log(set); // {1, "a", 4, 5}
console.log(set.size); // 4
console.log(set.has(1)); // true

set.forEach(function (value, key, ownerSet) {
  console.log(value);
  console.log(key);
});
// 1 1 a a 4 4 5 5
// 还有  keys()：返回键名。
// values()：返回键值。
// entries()：返回键值对
```

WeakSet 结构与 Set 类似，也是不重复的值的集合，WeakSet 的成员只能是对象，而不能是其他类型的值。
WeakSet 中的对象都是弱引用，即垃圾回收机制不考虑 WeakSet 对该对象的引用。也就是说 WeakSet 里面的引用，都不计入垃圾回收机制的引用计数。
没有 size 属性，不能 forEach，keys，values。不能遍历。其他与 Set 一样。

#### Map

Map 数据结构。它类似于对象，也是键值对的集合，但是“键”的范围不限于字符串，各种类型的值（包括对象）都可以当作键。

```javascript
let obj = {};
const map = new Map([
  [0, "张三"],
  [true, "Author"],
  [{}, { a: "aaa" }],
  [obj, { a: "aaa" }],
]);

console.log(map); // {0 => "张三", true => "Author", {…} => {…}}
map.size; // 4
map.has({}); // false
map.has(obj); // true
map.get(true); // Author

map.forEach((value, key, ownerMap) => {
  console.log(value);
  console.log(key);
});

//  张三
//  0
//  Author
//  true
//  {a: "aaa"}
//  {}
```

WeakMap 只接受对象作为键名（null 除外），不接受其他类型的值作为键名。
键名所引用的对象都是弱引用，即垃圾回收机制不将该引用考虑在内。（只针对键名）。
WeakMap 没有遍历操作。没有 size 属性

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

- target 目标对象，就是你要代理的对象。包括原生数组，函数，甚至另一个代理
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

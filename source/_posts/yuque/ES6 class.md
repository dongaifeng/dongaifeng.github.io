---
title: ES6 class
urlname: dsiraq
date: '2020-09-08 19:00:22 +0800'
tags: []
categories: []
---

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

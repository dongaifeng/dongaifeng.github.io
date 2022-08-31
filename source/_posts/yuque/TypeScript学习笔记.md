---
title: TypeScript学习笔记
urlname: by8nl5
date: '2021-05-28 10:12:11 +0800'
tags: []
categories: []
---

## 元组

元组就是长度一定，每个元素一定的数组。

```typescript
let x: [string, number];
x = ["hello", 10];
x[1] = "word"; // error
```

## 枚举

使用场景：http 状态码   比如 200 代表 success，获取 success 更易读

```typescript
enum Color {
  Red,
  Blue,
}
let a = Color.Red;
console.log(a); // 0 返回的值是下标 默认从0开始 也可以自己设置
enum Color2 {
  Yello,
  Red = "abc",
  Blue = 222,
  Green,
}
console.log(Color2.Green); // 223  下标会递增
console.log(Color2.Yello); // 0
```

object 表示非原始类型，也就是除`number`，`string`，`boolean`，`symbol`，`null`或`undefined`之外的类型

```typescript
function func(val: object | null): void {}
func(null);
func({});
func(222); // error
```

## 断言

```typescript
let data: any = { a: "aaa" };
// 假如data是ajax的数据 我不知道它是什么类型 但我可以猜测他是{a: string}
let _a: string = (<{ a: string }>data).a;
let _b: string = (data as { a: string }).a;
```

## 类型别名

类型别名会给一个类型起个新名字，和接口很像，可以作用于 原始值， 联合类型，元组

```typescript
type Name = string;
type Fn = () => string;
type NameOrFn = Name | Fn; // 是Name类型或者Fn类型

// 使用
function getName(prop: NameOrFn): Name {
  if (typeof prop === "string") {
    return prop;
  } else {
    return prop();
  }
}

// 类型别名 + 泛型 使用
type Con<T> = { val: T };

// 类型也可使用类型本身，类似递归
type Tree<T> = {
  val: T;
  child: Tree<T>;
};

// 类型别名 + 交叉类型
type List<T> = T & { next: List<T> };
// 使用
interface Person {
  name: string;
}

var people: List<Person>;
var name = people.next.next.name;
```

## 泛型

在定义函数、接口或类的时候，不预先指定具体的类型，而在使用的时候再指定类型的一种特性。
也可以不手动传入类型，它会根据你调用函数时传的实参去推断类型。

#### 一个简单的例子

一个传见指定长度的数组，用默认值来填充

```typescript
function createArray<T>(length: number, value: T): Array<T> {
  let result: T[] = [];
  for (let i = 0; i < length; i++) {
    result[i] = value;
  }
  return result;
}

createArray<string>(3, "x"); // ['x', 'x', 'x']
```

在函数名后添加了 <T>，T 用来表示函数调用时输入的类型，在后面的输入 value: T 和输出 Array<T> 中使用了 T。
可以看作 T 就是一个变量，代表了类型。
接着在调用的时候，指定它的类型为 string。
当然，也可以不手动指定，而让类型推论自动推算出来。

#### 多个类型参数

这个函数用来交换数组的元素

```typescript
function swap<T, U>(tuple: [T, U]): [U, T] {
  return [tuple[1], tuple[0]];
}

swap([7, "seven"]); // ['seven', 7]
```

#### 泛型约束

下面的例子，我们不能确定用户传进来的类型 T 有没有 length 属性，但是我们函数内部使用了 length。
那么，我们可以使用 extends 约束泛型，指定参数里面必须有 length 属性

```typescript
interface HaveLength {
  length: number;
}

function loggingIdentity<T extends HaveLength>(arg: T): T {
  console.log(arg.length);
  return arg;
}
```

#### 泛型接口，泛型类

```typescript
// 泛型类
class minClass<T> {
  public list: T[] = [];
  add(num: T) {
    this.list.push(num);
  }
}
let m1 = new minClass<number>(); // 调用时 传入T的类型
m1.add(11);
m1.add("22"); // error

// 泛型接口 一个函数类型 表示 一个函数只有一个参数val类型为T，返回值为string类型
interface fn {
  <T>(val: T): string;
}
var getData2: fn = function <T>(val: T): string {
  return val + "aaa";
};
getData2<string>("bbb");
getData2<string>(222); // error

// 还可以这样 定义一个函数的形状
interface fn2<T> {
  (value: T): T;
}

function Fn<T>(value: T): T {
  return value;
}
let myFn: fn2<string> = Fn;
myFn("20");

// 用类 作为 泛型的参数
class User {
  name: string | undefined;
  code: string | undefined;
}
class MySql<T> {
  add(info: T) {
    console.log(info);
  }
}
let db = new MySql<User>();
let u = new User();
u.name = "aaa";
u.code = "111";
db.add(u);
db.add({ name: "bbb", code: "222", pwd: "222" }); // error
let obj = { name: "bbb", code: "222", pwd: "222" };
db.add(obj);

// 设置泛型默认类型
function createArray<T = string>(value: T): Array<T> {
  return [value];
}
```

## 装饰器

```typescript
// 装饰器是一种特殊的类型声明，其实就是一个函数
function logClass(params: any) {
  console.log(params); // params 就是myHttp
  params.prototype.desc = "给类添加属性";
  params.prototype.add = function () {
    console.log("给类添加方法");
  };
}

@logClass
class myHttp {}

// 装饰器工厂 可以传参
function logClass2(params: any) {
  return function (target: any) {
    console.log(params); // params 是装饰器工厂传进来参数 return的才是真正的装饰器
    console.log(target); // target 是要装饰的类
    target.prototype.desc = params;
  };
}

@logClass2("hello")
class Color {}

// 重写构造函数
function newClass(target: any) {
  console.log(target);
  return class extends target {
    desc: any = "新的class 属性"; // 重写了 desc方法
    name: any | undefined;
    constructor() {
      super();
      this.name = "添加了属性"; // 添加了name属性
    }
    getData() {
      console.log("newClass 的方法");
    } // 重写了 getData方法
  };
}

@newClass
class oldClass {
  public desc: any | undefined;
  constructor() {
    this.desc = "oldClass 的属性";
  }
  getData() {
    console.log("oldClass 的方法");
  }
}

// 属性装饰器
function logProp(params: any) {
  return function (target: any, attr: any) {
    console.log(target); // target是原型的对象
    console.log(attr); // 要修饰的属性名称
    target[attr] = params;
  };
}

class MyClass {
  @logProp("属性")
  public desc: string | undefined;
  getData() {}
}

// 方法装饰器
function get(params: any) {
  return function (target: any, attr: any, desc: any) {
    console.log(target); // 原型对象
    console.log(attr); // 方法名称
    console.log(desc); // 属性描述符
    let _getData = desc.value; // 保存原来的getData方法
    // 重新定义getData
    desc.value = function (...args: any[]) {
      console.log(args);
      _getData.apply(this, [...args, params]); // 调用原来的方法，并设置上下文为原来的，将参数放回去
    };
  };
}

class myClass2 {
  public url: string | undefined;
  constructor() {
    this.url = "api";
  }
  @get("proxy")
  getData(...args: any[]) {
    console.log(args);
    console.log(this.url);
  }
}

let obj = new myClass2();
obj.getData(111, 222);

// 装饰器的执行顺序
// 属性>方法>方法参数>类
// 如果有多个相同的装饰器，先执行后面的
```

## TS 中 class 的修饰符

```typescript
// ts中class的三种修饰符
// public 公有 在当前类里面， 子类， 类外面都可访问 ,默认类型
// protected 受保护类型  在当前类里面， 子类里面可访问
// private 私有 当前类里面可访问

class Person {
  public a: string | undefined = "aaa";
  protected b: string | undefined = "bbb";
  private c: string | undefined = "ccc";
  constructor() {
    this.a = "aaa";
    this.b = "bbb";
    this.c = "ccc";
  }
  run() {
    console.log(this.a);
    console.log(this.b);
    console.log(this.c);
  }
}

class Son extends Person {
  work() {
    console.log(this.a);
    console.log(this.b); // 可以访问
    console.log(this.c); // error
  }
}

let person = new Person();
console.log(person.b); // 外部访问不了  error
console.log(person.c); // 外部访问不了  error

let son = new Son();
console.log(son.b); // 外部访问不了  error
console.log(son.c); // 外部访问不了  error

// 抽象类：它是提供其他类继承的基类，不能直接被实例化
// 用abstract关键字定义抽象类和抽象方法，抽象类中的抽象方法不包含具体实现并且必须在派生类中实现。
// abstract抽象方法只能放在抽象类里面
abstract class Animal {
  // 不能new
  name: string | undefined;
  constructor(name: string) {
    this.name = name;
  }
  run() {
    console.log(this.name);
  }
  abstract eat(foot: string): number; // 只能这样定义一下 不能实现功能 限时了参数，返回值
}

// new Animal() // error

class Dog extends Animal {
  constructor(name: string) {
    super(name);
  }
  eat(foot: any) {
    console.log(foot);
    return 2;
  }
}

let dog = new Dog("小黑");
dog.run();
dog.eat(2);
```

new  类的时候会调用 class 里面的 constructor。

## 接口

他是对   对象(其他类型的数据也可以)  各个属性的类型   的一种规定和描述

- 赋值的时候，变量的形状必须和接口的形状保持一致，多一个属性也不行。

```typescript
// 可选属性  age属性也可以没有
interface Person {
  name: string;
  age?: number;
}

// 任意属性
interface Person {
  name: string;
  age?: number;
  [propName: string]: any;
}

let tom: Person = {
  name: "Tom",
  gender: "male",
};
// 需要注意的是： 定义了任意属性，那其他的属性比如name，age的类型 必须在任意类型里也有
// 因为程序会把 其他属性都认为 是任意属性去校验
interface Person {
  name: string;
  age?: number;
  [propName: string]: string | number;
}

// 只读属性
interface Person {
  readonly id: number;
  [propName: string]: any;
}

// 接口继承
interface Animal {
  eat(): void; // 这是一个函数
}

interface Dog extends Animal {
  run(): void;
}

// implements实现接口
class MyDog implements Dog {
  eat() {}
  run() {}
}

// 对 数组/对象 的约束 基本没啥用
interface Arr {
  [index: number]: string;
}
interface Obj {
  [index: string]: string;
}

let arr3: Arr = ["1", "2"];
let obj: Obj = { 2: "aaa" };

// 对函数的约束, value属性可选
interface Fn {
  (key: string, code: number, value?: any): number;
}
let fn: Fn = function (a: string, b: number) {
  return b;
};

interface Person {
  readonly id: number; // 只读
  name: string;
  age?: number; // 可有可无
  [propName: string]: any; // 可定义任意属性，只要是string类型
}
```

implements  实现 ：
声明一个  class  类额时候使用   某个接口   就用到了  implements。

```typescript
interface Alarm {
  alert(): void;
}

interface Light {
  lightOn(): void;
  lightOff(): void;
}

// Car 这个类 实现了 两个interface，所以它里面必须有 两个接口里的属性
class Car implements Alarm, Light {
  alert() {
    console.log("Car alert");
  }
  lightOn() {
    console.log("Car light on");
  }
  lightOff() {
    console.log("Car light off");
  }
}

// SecurityDoor这个类 继承了Door，但是有实现了 Alarm这个接口
class SecurityDoor extends Door implements Alarm {
  alert() {
    console.log("SecurityDoor alert");
  }
}
```

接口不光能继承接口，也是可以继承类的，就是把一个类当成接口来用。

```typescript
class Point {
  x: number;
  y: number;
  constructor(x: number, y: number) {
    this.x = x;
    this.y = y;
  }
}
// 把 Point 当做一个 interface 来使用
interface Point3d extends Point {
  z: number;
}

let point3d: Point3d = { x: 1, y: 2, z: 3 };

// 也可直接把他当作 类型 来使用
function printPoint(p: Point) {
  console.log(p.x, p.y);
}
```

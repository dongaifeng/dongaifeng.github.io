---
title: TS学习2
urlname: ivuiko
date: '2020-09-08 19:00:22 +0800'
tags: []
categories: []
---

### 元组

元组就是长度一定，每个元素一定的数组。

```typescript
let x: [string, number];
x = ["hello", 10];
x[1] = "word"; // error
```

### 枚举

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

object  表示非原始类型，也就是除`number`，`string`，`boolean`，`symbol`，`null`或`undefined`之外的类型

```typescript
function func(val: object | null): void {}
func(null);
func({});
func(222); // error
```

### 断言

```typescript
let data: any = { a: "aaa" };
// 假如data是ajax的数据 我不知道它是什么类型 但我可以猜测他是{a: string}
let _a: string = (<{ a: string }>data).a;
let _b: string = (data as { a: string }).a;
```

### 类型别名

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

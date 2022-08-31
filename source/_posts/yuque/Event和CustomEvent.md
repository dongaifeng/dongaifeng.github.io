---
title: Event和CustomEvent
urlname: etxif2
date: '2021-05-28 10:12:12 +0800'
tags: []
categories: []
---

## Event

Event()函数是 Event 对象的构造函数。

```javascript
/* 创建一个事件对象，名字为myEvent，事件类型为dog */
var myEvent = new Event("dog", {
  bubbles: true,
  cancelable: true,
  composed: true,
});

/* 给这个事件对象创建一个属性并赋值 */
myEvent.name = "新的事件！";

window.addEventListener("dog", function (e) {
  console.log(e);
});

/* 触发自定义事件 */
document.dispatchEvent(myEvent);
```

#### 语法

##### new Event(typeArg, eventInit);

- typeArg：指定事件类型，字符串。类似点击事件（click）等。
- eventInit：可选，传递 EventInit 类型的字典。实际上这个 EventInit 类型的字典也就是我们使用 InitEvent()时需要传递的参数，以键值对的形式传递，不过它可以多选一个参数：

bubbles：事件是否支持冒泡，boolean 类型，默认值为 false。
　　　 cancelable：是否可取消事件的默认行为，boolean 类型，默认值为 false。
　　　 composed：事件是否会触发 shadow DOM 根节点之外的事件监听器，boolean 类型，默认值为 false。

## CustomEvent

CustomEvent()函数是 CustomEvent 对象的构造函数。

```javascript
var myEvent2 = new CustomEvent("cat", {
  detail: {
    a: "你好",
  },
});

window.addEventListener("cat", function (e) {
  console.log(e);
});

// 触发该事件
if (window.dispatchEvent) {
  window.dispatchEvent(myEvent2);
} else {
  window.fireEvent(myEvent2);
}
```

#### 语法

##### new CustomEvent(typeArg, customEventInit);

CustomEventInit 字典也可以接受 EventInit 字典的参数。
另外：
可传递 detail 参数：可选，默认值为 null，类型为 any。这个值可用于数据传递。
**Event 和 CustomEvent 不支持 IE**
参考**：[https://www.cnblogs.com/cwsb/p/10384219.html](https://www.cnblogs.com/cwsb/p/10384219.html)**

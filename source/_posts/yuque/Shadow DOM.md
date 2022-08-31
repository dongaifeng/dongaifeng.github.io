---
title: Shadow DOM
urlname: odk1g8
date: '2021-05-28 10:12:10 +0800'
tags: []
categories: []
---

## Shadow DOM 是什么

- 直译过来就是影子 DOM：**他是独立封装的一段 html 代码块，他所包含的 html 标签、css 样式和 js 行为都可以隔离、隐藏起来。**
- 与 IFrame 有点类似，不过 IFrame 是另外一个独立的 html 页面，shadow DOM 是当前 html 页面的一个代码片段。
- 他是由[Web Components](https://developer.mozilla.org/zh-CN/docs/Web/Web_Components)里提出的一项技术，其他还有 Custom elements、HTML templates、HTML Imports 这些。
- shadow DOM 并不是一个特别新的概念，html 中的 video 标签就是使用 shadow DOM 的一个案例。使用它时，你在 html 只会看到一个 video 标签，但实际上播放器上还有一系列按钮和其他操作，这些就都是封装到 shadow dom 中的，对外界是不可见的。

## 原理是什么？

shadow DOM 是 Web Components 提供的一项技术，旨在构建组件应用，具有隔离 DOM，作用域 css 的功能，类似于封装的概念，比如 video 标签。

## 基本用法

```javascript
// 通过attachShadow创建一个shadow Root
// open表示外部可以获取 Shadow DOM。Element.shadowRoot
// shadowDom获取宿主dom，shadowDom.host
let shadowDom = document
  .getElementById("shadow")
  .attachShadow({ mode: "open" });
let pElement = document.createElement("p");
pElement.innerHTML = "hello world";
let styleElement = document.createElement("style");
styleElement.textContent = `
    p{color:red}
`;
shadowDom.appendChild(pElement);
shadowDom.appendChild(styleElement);
```

## 概念

- Shadow host：宿主节点，一个常规 DOM 节点，Shadow DOM 会被附加到这个节点上。
- Shadow tree：Shadow DOM 内部的 DOM 树。
- Shadow boundary：Shadow DOM 结束的地方，也是常规 DOM 开始的地方。
- Shadow root: Shadow tree 的根节点。

![](https://cdn.nlark.com/yuque/0/2021/png/462392/1614909307386-4dee3a3b-9c5c-45cf-9fbd-a1bbe065789b.png#height=512&id=YHn94&originHeight=512&originWidth=800&originalType=binary&size=0&status=done&style=none&width=800)
参考：[https://segmentfault.com/a/1190000019115050](https://segmentfault.com/a/1190000019115050)

## web components 简单使用

```javascript
// 使用自定义标签
<user-card image="https://semantic-ui.com/images/avatar2/large/kristy.png" name="哈哈哈" email="333333"></user-card>

// 定义模板
<template id="userCardTemplate">
  <style>
    :host {
      display: flex;
      align-items: center;
      width: 450px;
      height: 180px;
      background-color: #d4d4d4;
      border: 1px solid #d5d5d5;
      box-shadow: 1px 1px 5px rgba(0, 0, 0, 0.1);
      border-radius: 3px;
      overflow: hidden;
      padding: 10px;
      box-sizing: border-box;
      font-family: 'Poppins', sans-serif;
    }
    .image {
      flex: 0 0 auto;
      width: 160px;
      height: 160px;
      vertical-align: middle;
      border-radius: 5px;
    }
  </style>
  <img src="https://semantic-ui.com/images/avatar2/large/kristy.png" class="image">
  <div class="container">
    <p class="name">User Name</p>
    <p class="email">yourmail@some-email.com</p>
    <button class="button">Follow</button>
  </div>
</template>

// 创建自定义标签
<script>
  class UserCard extends HTMLElement {
    constructor() {
      super();
      var shadow = this.attachShadow( { mode: 'closed' } );

      var templateElem = document.getElementById('userCardTemplate');
      var content = templateElem.content.cloneNode(true);
      content.querySelector('img').setAttribute('src', this.getAttribute('image'));
      content.querySelector('.container>.name').innerText = this.getAttribute('name');
      content.querySelector('.container>.email').innerText = this.getAttribute('email');

      shadow.appendChild(content);
    }
  }

  window.customElements.define('user-card', UserCard);
</script>

```

参考：[http://www.ruanyifeng.com/blog/2019/08/web_components.html](http://www.ruanyifeng.com/blog/2019/08/web_components.html)

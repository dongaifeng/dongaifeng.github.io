---
title: 自定义react
urlname: trg0qh
date: '2020-09-08 18:59:45 +0800'
tags: [react]
categories: [学习笔记]
---

### React.js

```javascript
import { initVnode } from "./ReactDom";
//babel-loader 把jsx编译成js对象，然后通过createElement处理成vnode
function createElement(type, props, ...children) {
  let vtype; // 用于判断节点类型
  props.children = children;

  if (typeof type === "string") {
    vtype = 1;
  } else if (typeof type === "function") {
    vtype = type.isReactComponent ? 2 : 3;
  }

  // 返回虚拟dom
  return {
    type,
    vtype,
    props,
  };
}

class Component {
  static isReactComponent = {};
  constructor(props) {
    this.props = props;
    this.$cache = {}; // 保存parentNode， node， vnode
  }

  setState = (nextState, calback) => {
    this.state = {
      ...this.state,
      ...nextState,
    };

    console.log(this.state);
    this.forceUpdate();
  };

  forceUpdate = () => {
    let newVnode = this.render();
    let node = diff(this.$cache, newVnode);
    this.$cache = {
      ...this.$cache,
      vnode: newVnode,
      node,
    };
  };
}

const React = {
  createElement,
  Component,
};

function diff(cache, newVNode) {
  const { vnode, node, parentNode } = cache;
  const newNode = initVnode(newVNode, parentNode);
  parentNode.replaceChild(newNode, node);
  return newNode;
}

export default React;
```

### ReactDom.js

```javascript
function render(vnode, container) {
  // 将vnode  变成 node  并插入到 container
  // vnode ----> node
  let node = initVnode(vnode, container);
  //渲染的时候把真实的dom节点放到container里面去，diff的时候是更新
  container.appendChild(node);
}

const ReactDom = {
  render,
};

export default ReactDom;
// 最终生成 真是node
export function initVnode(vnode, container) {
  let node = null;
  let { vtype } = vnode;
  if (vtype === 1) {
    node = initHtmlNode(vnode, container);
  }
  if (!vtype) {
    node = initTxtNode(vnode, container);
  }

  if (vtype === 3) {
    node = initFunctionComponent(vnode, container);
  }

  if (vtype === 2) {
    node = initClassComponent(vnode, container);
  }

  return node;
}

// 处理标签
function initHtmlNode(vnode, container) {
  const { type, props } = vnode;
  const node = document.createElement(type);
  const { children, ...rest } = props;

  children.map((item) => {
    node.appendChild(initVnode(item, node));
  });

  Object.keys(rest).forEach((item) => {
    if (item === "className") {
      node.setAttribute("class", rest[item]);
    }

    if (item.slice(0, 2) === "on") {
      node.addEventListener("click", rest[item]);
    }
  });

  return node;
}

// 处理text节点
function initTxtNode(vnode, container) {
  return document.createTextNode(vnode);
}
// 处理函数组件
function initFunctionComponent(vnode, container) {
  const { type, props } = vnode;
  let vvnode = type(props);
  return initVnode(vvnode, container);
}
// 处理class组件
function initClassComponent(vnode, container) {
  const { type, props } = vnode;
  let Comp = new type(props);
  let vvnode = Comp.render();
  const node = initVnode(vvnode, container);
  let cache = {
    parentNode: container,
    vnode: vvnode,
    node,
  };
  Comp.$cache = cache;
  return node;
}
```

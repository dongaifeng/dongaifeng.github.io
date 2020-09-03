---
title: Diff
urlname: vs2das
date: 2020-08-27 17:01:41 +0800
tags: []
categories: []
---

## 作用

Diff 是 Vue 中最重要的一部分，作用就是根据 vnode 找出其中最少差异，以至于最小化的更新 DOM。

## 工具

理解流程之前，首先要弄清楚 patch 过程中用到的一些工具函数。

#### insert

插入节点，如果有兄弟节点就插入到兄弟节点之前，如果没有就插入到末尾。
**nodeOps** 是一个对象包，里面包含对 dom 操作的原生方法的封装。

```javascript
function insert(parent, elm, ref) {
  if (isDef(ref)) {
    nodeOps.insertBefore(parent, elm, ref);
  } else {
    nodeOps.appendChild(parent, elm);
  }
}
```

#### createElm

根据 vnode 创建节点，判断 vnode.tag 分别会创建标签节点，注释节点，文本节点，组件。并且最后会调用 insert 插入到父节点。
而且会调用 createChildren 递归创建子节点。

```javascript
function createElm(vnode, parentElm, refElm) {
  var children = vnode.children;
  var tag = vnode.tag;
  if (tag) {
    vnode.elm = document.createElement(tag);

    // 先把 子节点插入 vnode.elm，然后再把 vnode.elm 插入parent
    createChildren(vnode, children);

    //  插入DOM 节点
    insert(parentElm, vnode.elm, refElm);
  } else {
    vnode.elm = document.createTextNode(vnode.text);
    insert(parentElm, vnode.elm, refElm);
  }
}
```

#### addVnodes

addVnodes 用来批量调用 createElm 新建节点。

```javascript
// 根据 vnodes 批量创建节点，vnodes数组并不是都会创建，而是从 startIdx开始到 endIdx
function addVnodes(
  parentElm,
  refElm,
  vnodes,
  startIdx,
  endIdx,
  insertedVnodeQueue
) {
  for (; startIdx <= endIdx; ++startIdx) {
    createElm(
      vnodes[startIdx],
      insertedVnodeQueue,
      parentElm,
      refElm,
      false,
      vnodes,
      startIdx
    );
  }
}
```

#### removeNodes

批量删除节点，循环调用 removeNode

#### sameVnode

判断两个节点是否相同，用处非常大。判断 key，tag，是否相同，**判断是否存在 data，不判断 data 是否相同**。
如果 tag 是 input 还要去判断 input 的 type

```javascript
function sameVnode(a, b) {
  return (
    a.key === b.key &&
    ((a.tag === b.tag &&
      a.isComment === b.isComment &&
      isDef(a.data) === isDef(b.data) &&
      sameInputType(a, b)) ||
      (isTrue(a.isAsyncPlaceholder) &&
        a.asyncFactory === b.asyncFactory &&
        isUndef(b.asyncFactory.error)))
  );
}
```

#### createKeyToOldIdx

接收一个  children  数组，生成 key 与 index 索引对应的一个 map 表。
这个 map 表，把 vnode 的 key 作为属性名，而该  vnode  在 children  的位置（index） 作为   属性值。
例如 :

```
{
    "key_1":0,
    "key_2":1,
    "key_4":2
}
```

他的作用是，当正常遍历完后，新节点有剩余，需要新建了，但是我可以看看新 vnode 是否再旧 vnode 数组中存在，并且可以拿到它的位置，找到它，然后就可以复用它，而不必新建了。

```javascript
function createKeyToOldIdx(children, beginIdx, endIdx) {
  let i, key;
  const map = {};
  for (i = beginIdx; i <= endIdx; ++i) {
    key = children[i].key;
    if (isDef(key)) map[key] = i;
  }
  return map;
}
```

## 概念

### 三种树

新 vnode，旧 vnode，页面 dom 树。
旧 vnode 和页面 dom 树是一一对应的。
新 vnode 是更新后页面 dom 要变成的样子。
新旧 Vnode 比较就是找出要改变的地方。
新旧 Vnode 比较，并会不对这两个 Vnode 修改，而是找出差别，直接对页面 dom 树修改。

### 原则

1. 只有判断两个节点是相同节点，才会比较子节点，也就是 patchVnode。
1. 只有同层级的节点才进行比较
1. 能不动就不动，实在不行就动动，还不行就删除/创建。
1. 先找不需要移动的的相同节点，再找相同的节点但是需要移动的，最后找不到相同的了，就新建/删除。

### 流程

## 源码

### patch

```javascript
return function patch(oldVnode, vnode, hydrating, removeOnly) {
  // 1 如果没有新vnode直接 说明组件被销毁
  if (isUndef(vnode)) {
    if (isDef(oldVnode)) invokeDestroyHook(oldVnode); // 触发destory钩子
    return;
  }

  let isInitialPatch = false;
  const insertedVnodeQueue = [];
  // 经过上边过滤，到这里说明 新节点 存在
  // 2 如果没有旧vnode 说明是新增节点
  if (isUndef(oldVnode)) {
    createElm(vnode, insertedVnodeQueue);
  } else {
    // 3 新，旧节点都存在
    // 并且两个节点相同，进入 patchVnode
    // 不同，销毁旧节点，新建新节点
    if (sameVnode(oldVnode, vnode)) {
      patchVnode(oldVnode, vnode, insertedVnodeQueue, null, null, removeOnly);
    } else {
      // 4 用空node代替 旧vnode
      oldVnode = emptyNodeAt(oldVnode);

      // 4.1 创新新节点 设置好 父节点 兄弟节点
      createElm(
        vnode,
        insertedVnodeQueue,
        oldElm._leaveCb ? null : parentElm,
        nodeOps.nextSibling(oldElm)
      );

      // destroy old node
      if (isDef(parentElm)) {
        removeVnodes([oldVnode], 0, 0);
      } else if (isDef(oldVnode.tag)) {
        invokeDestroyHook(oldVnode);
      }
    }
  }

  return vnode.elm;
};
```

### patchVnode

### updateChildren

## 总结

Diff  比较的内核是  **节点复用**，所以 Diff  比较就是为了在 新旧节点中   找到  **相同的节点  ，**实在找不到才会去创建。
找也不是瞎找，只会在同一层级找，什么叫同层级？就是父节点相同。
**比较是为了修改 DOM  树，**新旧  Vnode  树是拿来比较的，页面 DOM  树是拿来根据比较结果修改的

## 与 React 比较

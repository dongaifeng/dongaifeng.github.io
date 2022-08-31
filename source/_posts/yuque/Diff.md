---
title: Diff
urlname: kfzqgy
date: '2020-09-08 18:58:37 +0800'
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

根据 vnode 创建节点，判断 vnode.tag 是什么， 分别会创建标签节点，注释节点，文本节点，组件。并且最后会调用 insert 插入到父节点。
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

### 简单例子

比如下图存在这两棵 需要比较的新旧节点树 和 一棵 需要修改的页面 DOM 树

![](https://cdn.nlark.com/yuque/0/2020/png/462392/1599186096238-4e43fa10-67c3-43bd-89e1-fe5519aea1c3.png#align=left&display=inline&height=386&margin=%5Bobject%20Object%5D&originHeight=386&originWidth=574&size=0&status=done&style=shadow&width=574)

#### 第一轮比较开始

因为父节点都是 1，所以开始比较他们的子节点
按照我们上面的比较逻辑，所以先找 相同 && 不需移动 的点
毫无疑问，找到 2

![](https://cdn.nlark.com/yuque/0/2020/png/462392/1599186169843-1446783f-b067-4636-8413-cbc7d11d06c3.png#align=left&display=inline&height=250&margin=%5Bobject%20Object%5D&originHeight=250&originWidth=609&size=0&status=done&style=shadow&width=609)

拿到比较结果，这里不用修改 DOM，所以 DOM 保留在原地

![](https://cdn.nlark.com/yuque/0/2020/png/462392/1599186169982-fb7e5a7e-0b37-4fed-871d-4f462dbf24c4.png#align=left&display=inline&height=226&margin=%5Bobject%20Object%5D&originHeight=226&originWidth=321&size=0&status=done&style=shadow&width=321)

#### 第二轮比较开始

然后，没有 相同 && 不需移动 的节点 了
只能第二个方案，开始找相同的点
找到 节点 5，相同但是位置不同，所以需要移动

![](https://cdn.nlark.com/yuque/0/2020/png/462392/1599186169959-f01ec873-af54-4a2d-be35-37a638cc47ff.png#align=left&display=inline&height=239&margin=%5Bobject%20Object%5D&originHeight=239&originWidth=589&size=0&status=done&style=shadow&width=589)

拿到比较结果，页面 DOM 树需要移动 DOM 了，不修改，原样移动

![](https://cdn.nlark.com/yuque/0/2020/png/462392/1599186169830-f86db53e-2905-4e1c-b791-5951fd664e30.png#align=left&display=inline&height=229&margin=%5Bobject%20Object%5D&originHeight=229&originWidth=336&size=0&status=done&style=shadow&width=336)

#### 第三轮比较开始

继续，哦吼，相同节点也没得了，没得办法了，只能创建了
所以要根据 新 Vnode 中没找到的节点去创建并且插入
然后旧 Vnode 中有些节点不存在 新 VNode 中，所以要删除

![](https://cdn.nlark.com/yuque/0/2020/png/462392/1599186170075-0601af4e-a7a0-4150-ad55-ec455e8bb109.png#align=left&display=inline&height=265&margin=%5Bobject%20Object%5D&originHeight=265&originWidth=620&size=0&status=done&style=shadow&width=620)

于是开始创建节点 6 和 9，并且删除节点 3 和 4

![](https://cdn.nlark.com/yuque/0/2020/png/462392/1599186169888-b83d9a14-11e0-441e-ad6d-95a52c3a7604.png#align=left&display=inline&height=374&margin=%5Bobject%20Object%5D&originHeight=374&originWidth=608&size=0&status=done&style=shadow&width=608)

然后页面就完成更新啦

### Diff 流程

现在 Vue 需要更新，存在下面两组新旧子节点，需要进行比较，来判断需要更新哪些节点

![](https://cdn.nlark.com/yuque/0/2020/png/462392/1599211661945-35dc8cf3-22bb-4f96-8ef0-e2b116051dd0.png#align=left&display=inline&height=136&margin=%5Bobject%20Object%5D&originHeight=136&originWidth=407&size=0&status=done&style=shadow&width=407)

##### 头头比较，节点一样，不需移动，只用更新索引

![](https://cdn.nlark.com/yuque/0/2020/png/462392/1599211661729-97519fb4-e63c-4e55-a381-8d3aecca8f06.png#align=left&display=inline&height=225&margin=%5Bobject%20Object%5D&originHeight=225&originWidth=423&size=0&status=done&style=shadow&width=423)

更新索引，newStartIdx++ ， oldStartIdx++
开始下轮处理

##### 一系列判断之后，【旧头 2】 和 【 新尾 2】相同，直接移动到 oldEndVnode 后面

![](https://cdn.nlark.com/yuque/0/2020/png/462392/1599211661936-5acab30c-0749-412e-a8eb-36dd7fbeb66a.png#align=left&display=inline&height=370&margin=%5Bobject%20Object%5D&originHeight=370&originWidth=508&size=0&status=done&style=shadow&width=508)

更新索引，newEndIdx-- ，oldStartIdx ++
开始下轮处理

##### 一系列判断之后，【旧头 2】 和 【 新尾 2】相同，直接移动到 oldStartVnode 前面

![](https://cdn.nlark.com/yuque/0/2020/png/462392/1599211661723-3e80b0d4-9348-463d-a750-a918893aeff9.png#align=left&display=inline&height=370&margin=%5Bobject%20Object%5D&originHeight=370&originWidth=496&size=0&status=done&style=shadow&width=496)

更新索引，oldEndIdx-- ，newStartIdx++
开始下轮比较

##### 只剩一个节点，走到最后一个判断，单个查找

找不到一样的，直接创建插入到 oldStartVnode 前面

![](https://cdn.nlark.com/yuque/0/2020/png/462392/1599211661784-1d64620e-a6e9-4aa6-9cf4-b734c994b893.png#align=left&display=inline&height=358&margin=%5Bobject%20Object%5D&originHeight=358&originWidth=569&size=0&status=done&style=shadow&width=569)

更新索引，newStartIdx++
此时 newStartIdx> newEndIdx ，结束循环

##### 批量删除可能剩下的老节点

此时看 旧 Vnode 数组中， oldStartIdx 和 oldEndIdx 都指向同一个节点，所以只用删除 oldVnode-4 这个节点
ok，完成所有比较流程

## 源码

当数据改变，会触发组件更新，生成新的 Vnode，调用\_update。进行 patch。patch 函数是由 createPatchFunction 函数返回的，巧妙的利用了闭包。

### patch

这个函数主要是判断 oldVnode, vnode。
主要步骤是：

1. 没有新节点
1. 有新节点，没有旧节点
1. 旧节点 和 新节点 自身一样（不包括其子节点）
1. 旧节点 和 新节点 自身不一样

```javascript
return function patch(oldVnode, vnode, hydrating, removeOnly) {
  // 1 如果没有新vnode 直接return 说明组件被销毁
  if (isUndef(vnode)) {
    if (isDef(oldVnode)) invokeDestroyHook(oldVnode); // 触发destory钩子
    return;
  }

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

      // 4.2 销毁旧节点
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

根据上面的源码，我们很容易看到每步做的事情，下面我们逐步分析

##### 没有新节点

没有新节点，说明新节点被销毁，下面就看有没有旧节点，有的话就销毁，没有就不做处理。

##### 有新节点，没有旧节点

没有旧节点，说明是页面刚开始初始化的时候，此时，根本不需要比较了，直接全部都是新建，所以只调用 createElm。

##### 旧节点 和 新节点 自身一样

旧节点 和 新节点自身一样时，直接调用 patchVnode，比较子节点。

##### 旧节点 和 新节点 自身不一样

当两个节点不一样的时候，不难理解，直接创建新节点，删除旧节点

### patchVnode

这个函数主要是判断新，旧 Vnode 的子节点是文本，还是标签数组。
主要流程：

1. 新 Vnode 是不是文本节点
1. 新 Vnode 不是文本节点，对比子节点
   1. 新旧 Vnode 都有子节点
   1. 只存在新子节点
   1. 只存在 旧子节
   1. 新节点和旧节点都不存在子节点，但是旧节点是文本

```javascript
function patchVnode(
  oldVnode,
  vnode,
  insertedVnodeQueue,
  ownerArray,
  index,
  removeOnly
) {
  const elm = (vnode.elm = oldVnode.elm);
  const oldCh = oldVnode.children;
  const ch = vnode.children;

  // 新节点不是文本
  if (isUndef(vnode.text)) {
    // 都有子节点，而且不一样  进入 updateChildren
    if (isDef(oldCh) && isDef(ch)) {
      if (oldCh !== ch)
        updateChildren(elm, oldCh, ch, insertedVnodeQueue, removeOnly);

      // 分别判断 只存在新子节点或者 旧子节点的情况
    } else if (isDef(ch)) {
      // 只存在新子节点，如过旧子节点是文本，就制空。然后添加新子节点
      if (isDef(oldVnode.text)) nodeOps.setTextContent(elm, "");
      addVnodes(elm, null, ch, 0, ch.length - 1, insertedVnodeQueue);

      // 只存在 旧子节，删除
    } else if (isDef(oldCh)) {
      removeVnodes(oldCh, 0, oldCh.length - 1);

      // 新节点和旧节点都不存在子节点，但是旧节点是文本，制空
    } else if (isDef(oldVnode.text)) {
      nodeOps.setTextContent(elm, "");
    }
    // 新子节点是文本，比较文本
  } else if (oldVnode.text !== vnode.text) {
    nodeOps.setTextContent(elm, vnode.text);
  }
}
```

##### 新 Vnode 是不是文本节点

如果新 Vnode 的子节点是个文本，就和旧 Vnode 比较，更新。
如果不是文本，就进入判断子节点的流程

##### 对比子节点

新旧 Vnode 都有子节点的话，这一步判断不了，交给 updateChildren。
只有新子节点，说明旧节点没有，那就是新建啊，添加到父节点，就这个简单。
只有旧子节点，这个更好说，直接删除。
还有一种情况，就是旧字节点是文本，新节点没东西，那就直接设为空。

### updateChildren

这部分是最核心，也是最繁琐的地方。这是会使用 while 循环，而且设置四个游标。
分别是：
新子节点数组的第一个位置：简称：新头
新子节点数组的最后的位置：简称：新尾
旧子节点数组的第一个位置：简称：旧头
旧子节点数组的最后的位置：简称：新头

并且每个游标有它们自己对应位置的 Vnode。
循环的时候头部的游标会++，尾部的游标会--。从两头向中间一步一步各个击破。只要新子节点数组，旧子节点数组中有一组的头尾游标碰头了，循环旧结束。

比较的大概流程：

1. 旧头  ===  新头
1. 旧尾  ===  新尾
1. 旧头  ===  新尾
1. 旧尾  ===  新头
1. 新节点，单个拿出   去旧节点找相同的
1. 遍历完，剩下的
   1. 老节点群   先   结束  ：add 新节点群   剩下的
   1. 新节点群   先   结束  ：remove 老节点群   剩下的

![](https://cdn.nlark.com/yuque/0/2020/png/462392/1599190593565-1f501056-b56a-4bdd-a055-99130339640b.png#align=left&display=inline&height=271&margin=%5Bobject%20Object%5D&originHeight=271&originWidth=342&size=0&status=done&style=shadow&width=342)

```javascript
function updateChildren(
  parentElm,
  oldCh,
  newCh,
  insertedVnodeQueue,
  removeOnly
) {
  // 游标
  let oldStartIdx = 0;
  let newStartIdx = 0;
  let oldEndIdx = oldCh.length - 1;
  let newEndIdx = newCh.length - 1;

  // 游标对应的节点
  let oldStartVnode = oldCh[0];
  let oldEndVnode = oldCh[oldEndIdx];
  let newStartVnode = newCh[0];
  let newEndVnode = newCh[newEndIdx];

  let oldKeyToIdx, idxInOld, vnodeToMove, refElm;

  while (oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
    // 如果老头不存在 老头向前进一
    if (isUndef(oldStartVnode)) {
      oldStartVnode = oldCh[++oldStartIdx];

      // 如果老尾不存在 老头向前进一
    } else if (isUndef(oldEndVnode)) {
      oldEndVnode = oldCh[--oldEndIdx];

      // 如果老头 === 新头，拉去比较，老头，新头 都向前进一
    } else if (sameVnode(oldStartVnode, newStartVnode)) {
      patchVnode(
        oldStartVnode,
        newStartVnode,
        insertedVnodeQueue,
        newCh,
        newStartIdx
      );
      oldStartVnode = oldCh[++oldStartIdx];
      newStartVnode = newCh[++newStartIdx];

      // 如果老尾 === 新尾，拉去比较，老尾，新尾 都向前进一
    } else if (sameVnode(oldEndVnode, newEndVnode)) {
      patchVnode(
        oldEndVnode,
        newEndVnode,
        insertedVnodeQueue,
        newCh,
        newEndIdx
      );
      oldEndVnode = oldCh[--oldEndIdx];
      newEndVnode = newCh[--newEndIdx];

      // 这里Vue做个一个推测，推测用户是不是把子节点颠倒了
      // 所以会比较，旧头 === 新尾  旧尾 === 新头
    } else if (sameVnode(oldStartVnode, newEndVnode)) {
      // Vnode moved right
      patchVnode(
        oldStartVnode,
        newEndVnode,
        insertedVnodeQueue,
        newCh,
        newEndIdx
      );
      canMove &&
        nodeOps.insertBefore(
          parentElm,
          oldStartVnode.elm,
          nodeOps.nextSibling(oldEndVnode.elm)
        );
      oldStartVnode = oldCh[++oldStartIdx];
      newEndVnode = newCh[--newEndIdx];
    } else if (sameVnode(oldEndVnode, newStartVnode)) {
      // Vnode moved left
      patchVnode(
        oldEndVnode,
        newStartVnode,
        insertedVnodeQueue,
        newCh,
        newStartIdx
      );
      canMove &&
        nodeOps.insertBefore(parentElm, oldEndVnode.elm, oldStartVnode.elm);
      oldEndVnode = oldCh[--oldEndIdx];
      newStartVnode = newCh[++newStartIdx];
    } else {
      // 上面的比较都没有命中，那就把新头，拿出来去旧子节点数组找，有没有相同的
      // 找到了，那么只移动找到的那个节点就好了，所以要移动到老头前
      // 找不到旧新建，放到老头前
      if (isUndef(oldKeyToIdx))
        oldKeyToIdx = createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx);
      idxInOld = isDef(newStartVnode.key)
        ? oldKeyToIdx[newStartVnode.key]
        : findIdxInOld(newStartVnode, oldCh, oldStartIdx, oldEndIdx);
      if (isUndef(idxInOld)) {
        // New element
        createElm(
          newStartVnode,
          insertedVnodeQueue,
          parentElm,
          oldStartVnode.elm,
          false,
          newCh,
          newStartIdx
        );
      } else {
        vnodeToMove = oldCh[idxInOld];
        if (sameVnode(vnodeToMove, newStartVnode)) {
          patchVnode(
            vnodeToMove,
            newStartVnode,
            insertedVnodeQueue,
            newCh,
            newStartIdx
          );
          oldCh[idxInOld] = undefined;
          canMove &&
            nodeOps.insertBefore(parentElm, vnodeToMove.elm, oldStartVnode.elm);
        } else {
          // same key but different element. treat as new element
          createElm(
            newStartVnode,
            insertedVnodeQueue,
            parentElm,
            oldStartVnode.elm,
            false,
            newCh,
            newStartIdx
          );
        }
      }
      newStartVnode = newCh[++newStartIdx];
    }
  }

  // 处理剩下的节点
  if (oldStartIdx > oldEndIdx) {
    refElm = isUndef(newCh[newEndIdx + 1]) ? null : newCh[newEndIdx + 1].elm;
    addVnodes(
      parentElm,
      refElm,
      newCh,
      newStartIdx,
      newEndIdx,
      insertedVnodeQueue
    );
  } else if (newStartIdx > newEndIdx) {
    removeVnodes(oldCh, oldStartIdx, oldEndIdx);
  }
}
```

##### 旧头  ===  新头

两个新旧的两个头一样的时候，递归调用 patchVnode，比较它们的子节点
然后它们两个游标向前移动一位

##### 旧尾  ===  新尾

和上面的流程一样

##### 旧头  ===  新尾

这里说明旧头对应的这个 dom 现在已经移动到新尾对应的位置了。
将 旧头 移动到 旧尾的下一下兄弟节点的前面。
insertBefore(
parentElm,
oldStartVnode.elm,
nodeOps.nextSibling(oldEndVnode.elm)
)
![](https://cdn.nlark.com/yuque/0/2020/png/462392/1599209623480-b027f7e3-c038-4018-9155-bd01142ca919.png#align=left&display=inline&height=257&margin=%5Bobject%20Object%5D&originHeight=367&originWidth=691&size=0&status=done&style=shadow&width=483)

因为旧子节点数组和页面 dom 是一一对应的，而新尾和旧尾后面的节点肯定是比较过的，已经处理好的，所以可用通过旧尾找到新尾对应的位置。

##### 旧尾  ===  新头

和上面的是一样的
insertBefore(
parentElm,
oldEndVnode.elm,
oldStartVnode.elm
)
![](https://cdn.nlark.com/yuque/0/2020/png/462392/1599210178411-22a7f2da-f572-4db6-95e4-7ac2654bc5ba.png#align=left&display=inline&height=361&margin=%5Bobject%20Object%5D&originHeight=361&originWidth=494&size=0&status=done&style=shadow&width=494)

##### 单个查找

前面四种情况都不符合的话，只能一个一个查找了，目的就是不想新建，最大化复用旧的。
生成旧节点数组的 map 表，用新头节点的 key 值去 map 表匹配。
匹配到了，而且两个节点相同，就放到 旧头前面，为什么放到旧头前面？？？
因为使用新头匹配的，新头和旧头之前的肯定已经处理过了，dom 也已经处理好了，旧子节点数组和 dom 又是一一对应的，所以可以直接放到旧头前面。

##### 处理剩下的

当遍历完后，如果老节点数组先遍历完，新子节点数组有剩余，就挨个创建
如果新节点数组先遍历完，老节点有剩余，就挨个删除，包括他们的子节点。
![](https://cdn.nlark.com/yuque/0/2020/png/462392/1599211105511-c6e9b1a7-c5f0-499e-83d1-c9356f2194ea.png#align=left&display=inline&height=384&margin=%5Bobject%20Object%5D&originHeight=384&originWidth=622&size=0&status=done&style=shadow&width=622)

## 总结

Diff  比较的内核是  **节点复用**，所以 Diff  比较就是为了在 新旧节点中   找到  **相同的节点  ，**实在找不到才会去创建。
找也不是瞎找，只会在同一层级找，什么叫同层级？就是父节点相同。
**比较是为了修改 DOM  树，**新旧  Vnode  树是拿来比较的，页面 DOM  树是拿来根据比较结果修改的。

## 与 React 比较

即将更新

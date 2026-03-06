在更新过程中的 beginWork 阶段，分别执行各类tags的组件的 update函数，末尾都会调用 reconcileChildren 进入到 fiber的 diff 流程

diff的过程是 oldFiber 和 newChildren（jsx 转成的 虚拟dom）的对比，相同就复用fiber，否则就新建fiber，这就是 workInProgress的生成过程。生成的fiber上会打上标记，在commit阶段进行dom操作。

```typescript
export const Placement = /*                    */ 0b00000000000000000000000010; // 移动，新增
export const Update = /*                       */ 0b00000000000000000000000100; // 更新
export const Deletion = /*                     */ 0b00000000000000000000001000; // 删除
export const ChildDeletion = /*                */ 0b00000000000000000000010000; // 删除子节点
export const ContentReset = /*                 */ 0b00000000000000000000100000;
```

![画板](https://cdn.nlark.com/yuque/0/2026/jpeg/462392/1769070370841-dc067422-2fdf-43d7-b9fb-848fabd02062.jpeg)

子节点diff过程函数

```typescript
  function reconcileChildrenArray(
    returnFiber: Fiber,
    currentFirstChild: Fiber | null,
    newChildren: Array<*>,
    lanes: Lanes,
  ): Fiber | null {

    let resultingFirstChild: Fiber | null = null;  // 生成的第一个子节点
    let previousNewFiber: Fiber | null = null;     // 上一个 新节点

    let oldFiber = currentFirstChild;              // 旧的第一个子节点
    let lastPlacedIndex = 0;                       // 上一个被移动的节点的索引
    let newIdx = 0;                                // 新节点的索引
    let nextOldFiber = null;                       // 下一个旧节点

    // 第一遍循环，尝试将新节点和旧节点一一对应起来，如果一一对应不上，则跳出循环
    for (; oldFiber !== null && newIdx < newChildren.length; newIdx++) {
      if (oldFiber.index > newIdx) {
        nextOldFiber = oldFiber;
        oldFiber = null;
      } else {
        nextOldFiber = oldFiber.sibling;
      }
      
      // 通过对比 elementType，key 来判断旧节点是否可以复用
      // 可以，根据旧fiber，创建新fiber，并复用ref
      const newFiber = updateSlot( returnFiber, oldFiber, newChildren[newIdx], lanes,);

      // 没有找到 旧退出循环
      if (newFiber === null) { 
        if (oldFiber === null) {
          oldFiber = nextOldFiber;
        }
        break;
      }
      
      if (shouldTrackSideEffects) {
        if (oldFiber && newFiber.alternate === null) {
          // We matched the slot, but we didn't reuse the existing fiber, so we
          // need to delete the existing child.
          deleteChild(returnFiber, oldFiber);
        }
      }
      
      // 判断该节点是否需要移动，并返回最新的 lastPlacedIndex
      lastPlacedIndex = placeChild(newFiber, lastPlacedIndex, newIdx);
      
      if (previousNewFiber === null) {
        resultingFirstChild = newFiber;
      } else {
        previousNewFiber.sibling = newFiber;
      }
      previousNewFiber = newFiber;
      oldFiber = nextOldFiber;
    }

    // 新节点遍历完，剩余的旧节点全部删除
    if (newIdx === newChildren.length) {
      deleteRemainingChildren(returnFiber, oldFiber); // 标记删除节点
      return resultingFirstChild;
    }

    // 旧节点遍历完，新节点还有剩余，剩余的新节点全部创建
    if (oldFiber === null) {
      for (; newIdx < newChildren.length; newIdx++) {
        const newFiber = createChild(returnFiber, newChildren[newIdx], lanes);
        if (newFiber === null) {
          continue;
        }
        lastPlacedIndex = placeChild(newFiber, lastPlacedIndex, newIdx);
        if (previousNewFiber === null) {
          resultingFirstChild = newFiber;
        } else {
          previousNewFiber.sibling = newFiber;
        }
        previousNewFiber = newFiber;
      }
   
      return resultingFirstChild;
    }

    // 旧节点还有剩余，新节点也还有剩余 建立旧节点的map表，以便快速查找旧节点
    const existingChildren = mapRemainingChildren(returnFiber, oldFiber);

    // 第二遍循环，处理剩余的新节点
    for (; newIdx < newChildren.length; newIdx++) {
      // 尝试从旧节点map中获取对应的旧节点，通过key，type对比，并根据旧节点生成新节点，没有找到的会新建节点
      const newFiber = updateFromMap(
        existingChildren,
        returnFiber,
        newIdx,
        newChildren[newIdx],
        lanes,
      );

      // 可以复用的，map表中会删除对应的旧节点
      if (newFiber !== null) {在·
        if (shouldTrackSideEffects) {
          if (newFiber.alternate !== null) {
            existingChildren.delete(    // map表中会删除对应的旧节点
              newFiber.key === null ? newIdx : newFiber.key,
            );
          }
        }
        
        // 判断该节点是否需要移动，并返回最新的 lastPlacedIndex
        lastPlacedIndex = placeChild(newFiber, lastPlacedIndex, newIdx);

        // 更新 previousNewFiber
        if (previousNewFiber === null) {
          resultingFirstChild = newFiber;
        } else {
          previousNewFiber.sibling = newFiber;
        }
        previousNewFiber = newFiber;
      }
    }

    // 删除map表中剩余的节点
    if (shouldTrackSideEffects) {
      existingChildren.forEach(child => deleteChild(returnFiber, child));
    }

    return resultingFirstChild;
  }
```



对比节点

```typescript
  function updateSlot(
    returnFiber: Fiber,
    oldFiber: Fiber | null,
    newChild: any,
    lanes: Lanes,
  ): Fiber | null {

    const key = oldFiber !== null ? oldFiber.key : null;

    // 文本节点处理
    if ((typeof newChild === 'string' && newChild !== '') || typeof newChild === 'number') {
      if (key !== null) {
        return null;
      }
      return updateTextNode(returnFiber, oldFiber, '' + newChild, lanes);
    }

    //
    if (typeof newChild === 'object' && newChild !== null) {
      switch (newChild.$$typeof) {
        case REACT_ELEMENT_TYPE: {
          if (newChild.key === key) { // 比较key

            // 函数内会比较elementType，调用useFiber 创建新fiber，并复用老iber的 stateNode，tag，updateQueue等等
            return updateElement(returnFiber, oldFiber, newChild, lanes);
          } else {
            return null;
          }
        }
        case REACT_PORTAL_TYPE: 
          
        case REACT_LAZY_TYPE: {
          const payload = newChild._payload;
          const init = newChild._init;
          return updateSlot(returnFiber, oldFiber, init(payload), lanes);
        }
      }

      if (isArray(newChild) || getIteratorFn(newChild)) {
        if (key !== null) {
          return null;
        }

        return updateFragment(returnFiber, oldFiber, newChild, lanes, null);
      }

      throwOnInvalidObjectType(returnFiber, newChild);
    }
    return null;
  }
```

标记 Placement

```typescript
  function placeChild(
    newFiber: Fiber,
    lastPlacedIndex: number,
    newIndex: number,
  ): number {
    newFiber.index = newIndex;
    if (!shouldTrackSideEffects) {
      newFiber.flags |= Forked;
      return lastPlacedIndex;
    }
    const current = newFiber.alternate;
    if (current !== null) {
      const oldIndex = current.index;
      // 如果老节点的index小于上一个被移动的节点的index，说明该节点需要被移动
      // 否则说明该节点可以继续留在原地，不需要移动
      //
      if (oldIndex < lastPlacedIndex) {
        // This is a move.
        newFiber.flags |= Placement;  // 标记该节点需要被移动
        return lastPlacedIndex;
      } else {
        // This item can stay in place.
        return oldIndex;
      }
    } else {
      // This is an insertion.
      newFiber.flags |= Placement;
      return lastPlacedIndex;
    }
  }
```



```typescript
  function updateFromMap(
    existingChildren: Map<string | number, Fiber>,
    returnFiber: Fiber,
    newIdx: number,
    newChild: any,
    lanes: Lanes,
  ): Fiber | null {
    if (
      (typeof newChild === 'string' && newChild !== '') ||
      typeof newChild === 'number'
    ) {
      const matchedFiber = existingChildren.get(newIdx) || null;
      return updateTextNode(returnFiber, matchedFiber, '' + newChild, lanes);
    }

    if (typeof newChild === 'object' && newChild !== null) {
      switch (newChild.$$typeof) {
        case REACT_ELEMENT_TYPE: {
          // 从map中根据key获取对应的旧节点，判断key，index是否相等，不相等则返回null
          const matchedFiber =   
            existingChildren.get(
              newChild.key === null ? newIdx : newChild.key,
            ) || null;
          return updateElement(returnFiber, matchedFiber, newChild, lanes);
        }
        case REACT_PORTAL_TYPE: 
  
      }

      if (isArray(newChild) || getIteratorFn(newChild)) {
        const matchedFiber = existingChildren.get(newIdx) || null;
        return updateFragment(returnFiber, matchedFiber, newChild, lanes, null);
      }

      throwOnInvalidObjectType(returnFiber, newChild);
    }

    return null;
  }
```

删除子节点，打标记 ChildDeletion，收集deletions

```typescript
  function deleteChild(returnFiber: Fiber, childToDelete: Fiber): void {
    if (!shouldTrackSideEffects) {
      return;
    }
    const deletions = returnFiber.deletions;
    if (deletions === null) {
      returnFiber.deletions = [childToDelete];
      returnFiber.flags |= ChildDeletion;
    } else {
      deletions.push(childToDelete);
    }
  }

  function deleteRemainingChildren(
    returnFiber: Fiber,
    currentFirstChild: Fiber | null,
  ): null {
    if (!shouldTrackSideEffects) {
      return null;
    }

    let childToDelete = currentFirstChild;
    while (childToDelete !== null) {
      deleteChild(returnFiber, childToDelete);
      childToDelete = childToDelete.sibling;
    }
    return null;
  }
```



title: React的hook原理
---


<font style="color:rgb(89, 89, 89);">hook规则：不能在循环，条件判断，嵌套函数中调用hook，只能函数顶层调用。</font>

<font style="color:rgb(89, 89, 89);">在每个fiber 节点上有一个属性</font>**<font style="color:rgb(142, 0, 75);">memoizedState </font>**保存这本节点的hook单向链表，react用链表的顺序来指定hook。

### hook的类型
```typescript
export type Hook = {
  memoizedState: any, // 不同hook，取值不同，useState、useReducer存state，useEffect，useLayoutEffect存effect单向循环链表
  baseState: any,
  baseQueue: Update<any, any> | null,
  queue: any,  // 当前hook的更新队列
  next: Hook | null, // 下一个hook指针，为null，表示最后一个hook
  };

// 函数组件有单独定义的Update， UpdateQueue
export type Update<S, A> = {
  lane: Lane,
  action: A,
  hasEagerState: boolean,
  eagerState: S | null,
  next: Update<S, A>,
};

export type UpdateQueue<S, A> = {
  pending: Update<S, A> | null,
  lanes: Lanes,
  dispatch: (A => mixed) | null,
  lastRenderedReducer: ((S, A) => S) | null,
  lastRenderedState: S | null,
};

```

<font style="color:rgb(89, 89, 89);"></font>

组件进入`render`阶段，根据tags标记分别处理，updateFunctionComponent处理函数组件，其中会调用`renderWithHooks`开始hook的处理

![画板](https://cdn.nlark.com/yuque/0/2026/jpeg/462392/1768549637965-1c9fdd27-f56e-41fd-9057-12e61371c5a9.jpeg)

hook的内存表现

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/462392/1768895605801-9f54d965-3ec9-4d83-96df-270c9a160e01.png)



```javascript
export function renderWithHooks<Props, SecondArg>(
  current: Fiber | null,
  workInProgress: Fiber,
  Component: (p: Props, arg: SecondArg) => any,
  props: Props,
  secondArg: SecondArg,
  nextRenderLanes: Lanes,
): any {
  renderLanes = nextRenderLanes;
  currentlyRenderingFiber = workInProgress; // 全局变量，记录当前fiber节点，当此函数组件处理完会清空



  workInProgress.memoizedState = null; // 初始化 hooks 单向链表
  workInProgress.updateQueue = null;
  workInProgress.lanes = NoLanes;


  // 全局变量， 所有hook的实现函数，区分mount和update
  ReactCurrentDispatcher.current =    
      current === null || current.memoizedState === null
        ? HooksDispatcherOnMount
        : HooksDispatcherOnUpdate;
  

  let children = Component(props, secondArg);  // 执行函数

  ReactCurrentDispatcher.current = ContextOnlyDispatcher;

  renderLanes = NoLanes; // 重置全局变量
  currentlyRenderingFiber = (null: any);

  currentHook = null;
  workInProgressHook = null;

  return children;
}
```

初始化阶段

```javascript
function mountReducer<S, I, A>(
  reducer: (S, A) => S,
  initialArg: I,
  init?: I => S,
): [S, Dispatch<A>] {
  // 创建hook对象,并赋值给fiber.memoizedState和全局变量workInProgressHook
  const hook = mountWorkInProgressHook(); 
  let initialState;
  // 计算初始 state
  if (init !== undefined) {
    initialState = init(initialArg);
  } else {
    initialState = ((initialArg: any): S);
  }
  hook.memoizedState = hook.baseState = initialState; // 初始state，赋值给hook对象的 memoizedState 和 baseState
  const queue: UpdateQueue<S, A> = {   // 创建queue，并赋值给hook.queue
    pending: null,
    lanes: NoLanes,
    dispatch: null,
    lastRenderedReducer: reducer,
    lastRenderedState: (initialState: any),
  };
  hook.queue = queue;
  const dispatch: Dispatch<A> = (queue.dispatch = (dispatchReducerAction.bind(
    null,
    currentlyRenderingFiber,
    queue,
  ): any));
  return [hook.memoizedState, dispatch];
}

function dispatchReducerAction<S, A>(
  fiber: Fiber,
  queue: UpdateQueue<S, A>,
  action: A,
) {
  const lane = requestUpdateLane(fiber); // 获取优先级

  const update: Update<S, A> = {  // 创建更新对象
    lane,
    action,
    hasEagerState: false,
    eagerState: null,
    next: (null: any),
  };

  if (isRenderPhaseUpdate(fiber)) {
    enqueueRenderPhaseUpdate(queue, update); 
  } else {
    // 更新对象暂存到concurrentQueues 数组中
    在render阶段finishQueueingConcurrentUpdates中，会加入到fiber的queue中
    const root = enqueueConcurrentHookUpdate(fiber, queue, update, lane);
    if (root !== null) {
      const eventTime = requestEventTime();
      // 调度更新
      scheduleUpdateOnFiber(root, fiber, lane, eventTime);
    }
  }
}
```

更新阶段

```typescript
function updateReducer<S, I, A>(
  reducer: (S, A) => S,
  initialArg: I,
  init?: I => S,
): [S, Dispatch<A>] {
  // 根据 当前hook，复制出一个新hook 并赋值给fiber.memoizedState和全局变量workInProgressHook
  const hook = updateWorkInProgressHook();  
  const queue = hook.queue; // hook上的 updateQueue

  queue.lastRenderedReducer = reducer;

  // queue.pending 赋值给 hook.baseQueue，然后将 queue.pending 置空
  const current: Hook = (currentHook: any);
  let baseQueue = current.baseQueue;

  const pendingQueue = queue.pending;
  if (pendingQueue !== null) {
    current.baseQueue = baseQueue = pendingQueue;
    queue.pending = null;
  }

  if (baseQueue !== null) {
    const first = baseQueue.next;
    let newState = current.baseState;

    let newBaseState = null;
    let newBaseQueueFirst = null;
    let newBaseQueueLast = null;
    let update = first;

    // 遍历baseQueue中的update对象，计算出新的state，并赋值给hook.memoizedState
    do {
 
      if (update.hasEagerState) {
        newState = ((update.eagerState: any): S); // 针对函数形式的setState
      } else {
        const action = update.action;
        newState = reducer(newState, action);
      }
      
      update = update.next;
    } while (update !== null && update !== first);

    if (newBaseQueueLast === null) {
      newBaseState = newState;
    } else {
      newBaseQueueLast.next = (newBaseQueueFirst: any);
    }

    // 更新hook状态，并返回
    hook.memoizedState = newState;
    hook.baseState = newBaseState;
    hook.baseQueue = newBaseQueueLast;
    queue.lastRenderedState = newState;
  }


  const dispatch: Dispatch<A> = (queue.dispatch: any);
  return [hook.memoizedState, dispatch];
}
```

### 副作用Effect
<font style="color:rgb(89, 89, 89);">调用</font>`<font style="color:rgb(255, 80, 44);background-color:rgb(255, 245, 245);">useEffect</font>`<font style="color:rgb(89, 89, 89);">hook实际上仅仅只是将相关信息保存在fiber上，然后打上标签。</font>

<font style="color:rgb(89, 89, 89);">真正执行副作用的时候是在</font>`<font style="color:rgb(255, 80, 44);background-color:rgb(255, 245, 245);">commit</font>`<font style="color:rgb(89, 89, 89);">阶段</font>

```typescript
export type Effect = {
  tag: HookFlags,
  create: () => (() => void) | void, // 回调函数
  destroy: (() => void) | void, // 销毁函数
  deps: Array<mixed> | null, // 依赖
  next: Effect, // 下一个effect
};
```

初始化阶段在 useEffect，useLayoutEffect，不仅会创建hook对象 加入到 fiber.memoizedState 的hook链表中。还会在fiber.updateQueue 添加此effect副作用

fiber.updateQueue对象只有两个属性一个是lastEffect, stores。 lastEffect指向最后一个effect，lastEffect.next 就是第一个effect。并且他是一个单向循环链表

```typescript
function mountLayoutEffect(
  create: () => (() => void) | void,
  deps: Array<mixed> | void | null,
): void {
  let fiberFlags: Flags = UpdateEffect;
  if (enableSuspenseLayoutEffectSemantics) {
    fiberFlags |= LayoutStaticEffect;
  }

  // HookLayout 表示此为useLayoutEffect
  return mountEffectImpl(fiberFlags, HookLayout, create, deps);
}

function mountEffect(
  create: () => (() => void) | void,
  deps: Array<mixed> | void | null,
): void {

    return mountEffectImpl(
      PassiveEffect | PassiveStaticEffect,
      HookPassive,  //  HookPassive表示此hook是useEffect
      create,
      deps,
    );
  
}

function mountEffectImpl(fiberFlags, hookFlags, create, deps): void {
  // 初始化创建hook 并赋值给 fiber.memoizedState
  const hook = mountWorkInProgressHook();
  const nextDeps = deps === undefined ? null : deps; // 依赖项
  currentlyRenderingFiber.flags |= fiberFlags;
  hook.memoizedState = pushEffect(
    HookHasEffect | hookFlags,
    create,
    undefined,
    nextDeps,
  );
}

function pushEffect(tag, create, destroy, deps) {
  // 创建effect对象
  const effect: Effect = { tag, create, destroy,  deps, next: (null: any), };
  
  // 获取fiber上的updateQueue
  let componentUpdateQueue: null | FunctionComponentUpdateQueue = (currentlyRenderingFiber.updateQueue: any);
  
  // 如果 updateQueue不存在就初始化 只有两个属性lastEffect, stores,
  // 并把当前effect挂载到lastEffect上
  if (componentUpdateQueue === null) {
    componentUpdateQueue = createFunctionComponentUpdateQueue();  
    currentlyRenderingFiber.updateQueue = (componentUpdateQueue: any);
    componentUpdateQueue.lastEffect = effect.next = effect;
    // 如果存在就加入到循环链表的最后
  } else {
    const lastEffect = componentUpdateQueue.lastEffect;
    if (lastEffect === null) {
      componentUpdateQueue.lastEffect = effect.next = effect;
    } else {
      const firstEffect = lastEffect.next;
      lastEffect.next = effect;
      effect.next = firstEffect;
      componentUpdateQueue.lastEffect = effect;
    }
  }
  return effect;
}
```

更新阶段，也是同样的流程，只是根据deps的变化，tags会有所不同

```typescript
function updateEffectImpl(fiberFlags, hookFlags, create, deps): void {
  // 根据当前hook,创建新hook, 并把新hook挂载到链表上
  const hook = updateWorkInProgressHook();
  const nextDeps = deps === undefined ? null : deps;
  let destroy = undefined;

  if (currentHook !== null) {
    const prevEffect = currentHook.memoizedState;  // 旧的 effect 对象
    destroy = prevEffect.destroy; // 旧的effect.destroy
    if (nextDeps !== null) { // 新的deps
      const prevDeps = prevEffect.deps;  // 旧的deps

      // 比较新旧deps, 如果相同, 就创建effect, 加入到 updateQueue,注意这里没有 HookHasEffect 标志
      if (areHookInputsEqual(nextDeps, prevDeps)) {
        hook.memoizedState = pushEffect(hookFlags, create, destroy, nextDeps);
        return;
      }
    }
  }

  currentlyRenderingFiber.flags |= fiberFlags;
// 这里是deps有变化的情况, 并且 flags 加入了 HookHasEffect
  hook.memoizedState = pushEffect(
    HookHasEffect | hookFlags,
    create,
    destroy,
    nextDeps,
  );
}
```



### 执行副作用
useEffect 被异步执行，用scheduler调度，并且是先执行destory，再执行 create。

layoutEffect也是在commit阶段被执行，render结束，浏览器绘制前。destory是在Mutation阶段，create是在Layout阶段。

```typescript
    // useEffect 被异步执行 
   scheduleCallback(NormalSchedulerPriority, () => {
        flushPassiveEffects();
        return null;
   });

  // 执行完 destory，立即执行 create
  commitPassiveUnmountEffects(root.current);
  commitPassiveMountEffects(root, root.current, lanes, transitions);
```

```typescript
function commitHookEffectListUnmount(  // 执行effect的 destory
  flags: HookFlags,
  finishedWork: Fiber,
  nearestMountedAncestor: Fiber | null,
) {
  // updateQueue保存了effect链表
  const updateQueue: FunctionComponentUpdateQueue | null = (finishedWork.updateQueue: any);
  const lastEffect = updateQueue !== null ? updateQueue.lastEffect : null;
  if (lastEffect !== null) {
    const firstEffect = lastEffect.next;
    let effect = firstEffect;
    do {
      if ((effect.tag & flags) === flags) {
        // Unmount
        const destroy = effect.destroy;
        effect.destroy = undefined;
        if (destroy !== undefined) {
       
          // 安全的执行destory
          safelyCallDestroy(finishedWork, nearestMountedAncestor, destroy);
        }
      }
      effect = effect.next;
    } while (effect !== firstEffect); // 循环链表退出条件，直到回到第一个节点
  }
}


function commitHookEffectListMount(flags: HookFlags, finishedWork: Fiber) {
  const updateQueue: FunctionComponentUpdateQueue | null = (finishedWork.updateQueue: any);
  const lastEffect = updateQueue !== null ? updateQueue.lastEffect : null;
  if (lastEffect !== null) {
    const firstEffect = lastEffect.next;
    let effect = firstEffect;
    do {
      if ((effect.tag & flags) === flags) {
  

        // Mount   执行effect的create函数
        const create = effect.create;
    
        effect.destroy = create();
      }
      effect = effect.next;
    } while (effect !== firstEffect);  // 循环链表退出条件，直到回到第一个节点
  }
}
```

### useCallback
```javascript
function mountCallback<T>(callback: T, deps: Array<mixed> | void | null): T {
  const hook = mountWorkInProgressHook();
  const nextDeps = deps === undefined ? null : deps;
  hook.memoizedState = [callback, nextDeps];
  return callback;
}

function updateCallback<T>(callback: T, deps: Array<mixed> | void | null): T {
  const hook = updateWorkInProgressHook();
  const nextDeps = deps === undefined ? null : deps;
  const prevState = hook.memoizedState;
  if (prevState !== null) {
    if (nextDeps !== null) {
      const prevDeps = prevState[1];
      // 比较新旧依赖， 全等返回旧的 callback，否则重新赋值 hook.memoizedState
      if (areHookInputsEqual(nextDeps, prevDeps)) {
        return prevState[0];
      }
    }
  }
  hook.memoizedState = [callback, nextDeps];
  return callback;
}


// 比较依赖  is内部只是 ===
function areHookInputsEqual(
  nextDeps: Array<mixed>,
  prevDeps: Array<mixed> | null,
) {
  for (let i = 0; i < prevDeps.length && i < nextDeps.length; i++) {
    if (is(nextDeps[i], prevDeps[i])) {
      continue;
    }
    return false;
  }
  return true;
}
```


































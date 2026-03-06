commit阶段是针对 finishedWork 每个节点fiber上的标记，做相应的处理，生命周期函数调用，hook的调用，dom操作，和标记，状态重置等。

render流程中beginWork给每个节点打flags，在`completeWork`的时候收集自己子节点上的`flags`作为自己的`subtreeFlags`，通过subtreeFlags 可以知道自己的子节点副作用的情况

```javascript
export type Flags = number;

export const NoFlags = /*                      */ 0b0000000000000000000000000000;
// 移动、新增
export const Placement = /*                    */ 0b0000000000000000000000000010;
// 更新
export const Update = /*                       */ 0b0000000000000000000000000100;
// 删除
export const ChildDeletion = /*                */ 0b0000000000000000000000010000;
// 内容重置
export const ContentReset = /*                 */ 0b0000000000000000000000100000;
// 回调
export const Callback = /*                     */ 0b0000000000000000000001000000;
// 引用
export const Ref = /*                          */ 0b0000000000000000001000000000;
// 快照
export const Snapshot = /*                     */ 0b0000000000000000010000000000;
// Hook
export const Passive = /*                      */ 0b0000000000000000100000000000;

// 生命周期相关
export const LifecycleEffectMask =
  Passive | Update | Callback | Ref | Snapshot ;


// 快照相关
export const BeforeMutationMask: number = Update | Snapshot 

// 改变，转变相关
export const MutationMask =
  Placement |
  Update |
  ChildDeletion |
  ContentReset |
  Ref 

// 布局相关
export const LayoutMask = Update | Callback | Ref 

// useEffect相关
export const PassiveMask = Passive | Visibility | ChildDeletion

```

React将commit阶段分为三个子阶段:

1. BeforeMutation: 处理带有`BeforeMutationMask`flags标记的副作用节点，class组件会调用getSnapshotBeforeUpdate钩子，
2. mutation: 处理副作用队列中`MutationMask`flag标记的副作用，主要包括 Placement ，Update ，ChildDeletion ，ContentReset ，Ref。主要工作包括：移动节点、删除节点、清除内容、添加节点；执行被删除组件的销毁钩子函数，执行被删除组件的`componentWillUnmount`；更新ref引用，执行layoutEffect的destory函数等等
3. Layout: 处理标记LayoutMask 包括：Update ，Callback ， Ref。 <font style="color:rgb(89, 89, 89);">执行 </font>`useLayoutEffect`的create，调用生命周期componentDidMount，componentDidUpdate，调用setState的callback

  

![画板](https://cdn.nlark.com/yuque/0/2026/jpeg/462392/1768900663536-54d5439f-e0a4-44b8-8d92-11035a3eff93.jpeg)

```javascript
function commitRootImpl(root,recoverableErrors,transitions, renderPriorityLevel) {
  do {
    flushPassiveEffects(); // 把上次渲染没有执行完的副作用执行完
  } while (rootWithPendingPassiveEffects !== null);

  const finishedWork = root.finishedWork;
  const lanes = root.finishedLanes;

  root.finishedWork = null; // 重置finishWork
  root.finishedLanes = NoLanes;

  root.callbackNode = null;
  root.callbackPriority = NoLane;


  let remainingLanes = mergeLanes(finishedWork.lanes, finishedWork.childLanes);
  const concurrentlyUpdatedLanes = getConcurrentlyUpdatedLanes();
  remainingLanes = mergeLanes(remainingLanes, concurrentlyUpdatedLanes);
  markRootFinished(root, remainingLanes);

  if (root === workInProgressRoot) {
    workInProgressRoot = null;
    workInProgress = null;
    workInProgressRootRenderLanes = NoLanes;
  } 

  // 异步执行本次commit的副作用
  if (
    (finishedWork.subtreeFlags & PassiveMask) !== NoFlags ||
    (finishedWork.flags & PassiveMask) !== NoFlags
  ) {
      scheduleCallback(NormalSchedulerPriority, () => { flushPassiveEffects(); });
  }


  const subtreeHasEffects =finishedWork.subtreeFlags & PassiveMask !== NoFlags;
  const rootHasEffect =finishedWork.flags & PassiveMask !== NoFlags;

  // 如果本轮 有副作用。开始mutation
  if (subtreeHasEffects || rootHasEffect) {

     // Before Mutation 阶段
    setCurrentUpdatePriority(DiscreteEventPriority);
    const shouldFireAfterActiveInstanceBlur = commitBeforeMutationEffects(               
      root,
      finishedWork,
    );

     // Mutation 阶段
    commitMutationEffects(root, finishedWork, lanes);                                     

    resetAfterCommit(root.containerInfo);

    // 完成
    root.current = finishedWork;

    // Layout 阶段
    commitLayoutEffects(finishedWork, root, lanes);                                                       

    // 请求浏览器绘制  直接调用Scheduler的requestPaint
    requestPaint();                                                                            

    setCurrentUpdatePriority(previousPriority);
  }

  const rootDidHavePassiveEffects = rootDoesHavePassiveEffects;

    /*
  * 每次commit阶段完成后，再执行一遍ensureRootIsScheduled，确保是否还有任务需要被调度。
  * 例如，高优先级插队的更新完成后，commit完成后，还会再执行一遍，保证之前跳过的低优先级任务
  * 重新调度
  *
  * */
  ensureRootIsScheduled(root, now());

  if (
    includesSomeLane(pendingPassiveEffectsLanes, SyncLane) &&
    root.tag !== LegacyRoot
  ) {
    flushPassiveEffects(); // 再检查一遍useEffect 有没有没执行的
  }

  flushSyncCallbacks(); //  再检查一遍setState的回调函数 有没有没执行的
}
```

### BeforeMutation阶段
```javascript
export function commitBeforeMutationEffects(
  root: FiberRoot,
  firstChild: Fiber,
) {
  focusedInstanceHandle = prepareForCommit(root.containerInfo);

  nextEffect = firstChild;
  commitBeforeMutationEffects_begin();

  // We no longer need to track the active instance fiber
  const shouldFire = shouldFireAfterActiveInstanceBlur;
  shouldFireAfterActiveInstanceBlur = false;
  focusedInstanceHandle = null;

  return shouldFire;
}

function commitBeforeMutationEffects_begin() {   
  while (nextEffect !== null) {
    const fiber = nextEffect;

    // This phase is only used for beforeActiveInstanceBlur.
    // Let's skip the whole loop if it's off.
    if (enableCreateEventHandleAPI) {
      // TODO: Should wrap this in flags check, too, as optimization
      const deletions = fiber.deletions;
      if (deletions !== null) {
        for (let i = 0; i < deletions.length; i++) {
          const deletion = deletions[i];
          commitBeforeMutationEffectsDeletion(deletion);
        }
      }
    }

    const child = fiber.child;
    if (
      (fiber.subtreeFlags & BeforeMutationMask) !== NoFlags &&
      child !== null
    ) {
      child.return = fiber;
      nextEffect = child;
    } else {
      commitBeforeMutationEffects_complete(); // 调用 commitBeforeMutationEffectsOnFiber
    }
  }
}
```



```javascript
function commitBeforeMutationEffectsOnFiber(finishedWork: Fiber) {
  const current = finishedWork.alternate;
  const flags = finishedWork.flags;

  if ((flags & Snapshot) !== NoFlags) {
    setCurrentDebugFiberInDEV(finishedWork);

    switch (finishedWork.tag) {
 
      case ClassComponent: {
        if (current !== null) {
          const prevProps = current.memoizedProps;
          const prevState = current.memoizedState;
          const instance = finishedWork.stateNode;
   
          const snapshot = instance.getSnapshotBeforeUpdate(  // 调用生命周期钩子
            finishedWork.elementType === finishedWork.type
              ? prevProps
              : resolveDefaultProps(finishedWork.type, prevProps),
            prevState,
          );
  
          instance.__reactInternalSnapshotBeforeUpdate = snapshot;
        }
        break;
      }
    }

  }
}
```

### mutation 阶段
```javascript
export function commitMutationEffects(root,finishedWork,committedLanes) {
  commitMutationEffectsOnFiber(finishedWork, root, committedLanes);
}
```

```typescript
function commitMutationEffectsOnFiber(
  finishedWork: Fiber,
  root: FiberRoot,
  lanes: Lanes,
) {
  const current = finishedWork.alternate;
  const flags = finishedWork.flags;
  switch (finishedWork.tag) {
    case FunctionComponent:
    case ForwardRef:
    case MemoComponent:
    case SimpleMemoComponent: {
      recursivelyTraverseMutationEffects(root, finishedWork, lanes);
      commitReconciliationEffects(finishedWork);

      if (flags & Update) {
        commitHookEffectListUnmount(  // 执行effect的 destory函数
          HookInsertion | HookHasEffect,
          finishedWork,
          finishedWork.return,
        );
        commitHookEffectListMount(  // 执行effect的 create函数
          HookInsertion | HookHasEffect,
          finishedWork,
        );
      }
      return;
    }
    case ClassComponent: {
      recursivelyTraverseMutationEffects(root, finishedWork, lanes);
      commitReconciliationEffects(finishedWork);
      return;
    }
    case HostComponent: 
    case HostText: 
    case HostRoot:
    case HostPortal: 
    case SuspenseComponent: 
    case OffscreenComponent:
    case SuspenseListComponent: 
    case ScopeComponent:
    default: {
      recursivelyTraverseMutationEffects(root, finishedWork, lanes);
      commitReconciliationEffects(finishedWork);
      return;
    }
  }
}
```

### <font style="color:rgb(25, 27, 31);">layout阶段</font>
```javascript
export function commitLayoutEffects(root,finishedWork,committedLanes) {
  commitLayoutEffects_begin(finishedWork, root, committedLanes);
}

function commitLayoutEffects_begin(
  subtreeRoot: Fiber,
  root: FiberRoot,
  committedLanes: Lanes,
) {
  while (nextEffect !== null) {
    const fiber = nextEffect;
    const firstChild = fiber.child;

    if (
      enableSuspenseLayoutEffectSemantics &&
      fiber.tag === OffscreenComponent &&
      isModernRoot
    ) {
      // Keep track of the current Offscreen stack's state.
      const isHidden = fiber.memoizedState !== null;
      const newOffscreenSubtreeIsHidden = isHidden || offscreenSubtreeIsHidden;
      if (newOffscreenSubtreeIsHidden) {
        // The Offscreen tree is hidden. Skip over its layout effects.
        commitLayoutMountEffects_complete(subtreeRoot, root, committedLanes);
        continue;
      } else {

        let child = firstChild;
        while (child !== null) {
          nextEffect = child;
          commitLayoutEffects_begin(
            child, // New root; bubble back up to here and stop.
            root,
            committedLanes,
          );
          child = child.sibling;
        }

        commitLayoutMountEffects_complete(subtreeRoot, root, committedLanes);
        continue;
      }
    }

    if ((fiber.subtreeFlags & LayoutMask) !== NoFlags && firstChild !== null) {
      firstChild.return = fiber;
      nextEffect = firstChild;
    } else {
      commitLayoutMountEffects_complete(subtreeRoot, root, committedLanes);
    }
  }
}
```


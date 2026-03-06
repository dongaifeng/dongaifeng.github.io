  版本：18.3.1

react的初始化流程包括：createRoot，render两部分

```javascript
// 第一步：创建root
const root = ReactDOM.createRoot(document.getElementById("root"));
// 第二步：渲染
root.render(<App />);
```

### createRoot
createRoot是为 render 阶段做准备

**创建root节点**

fiberRoot全局唯一单例，root.current 指向当前正在展示fiber，workInProgress指向更新时创建的fiber。这就是双缓存机制

containerInfo: 创建当前节点时的dom节点

**创建HostRootFiber**

hostRootFiber 为根fiber，每次更新都是从根节点开始

**建立FiberRoot与HostRootFiber关系   **

**并且会在root根节点上绑定所有的事件**

    流程图

![画板](https://cdn.nlark.com/yuque/0/2026/jpeg/462392/1768275827339-9520d84f-af4e-4d5f-bd62-591c9078f0fa.jpeg)

初始函数

```javascript
export function createRoot(container,options): RootType {

// 创建容器
  const root = createContainer(
    container,
    ConcurrentRoot,
    null,
    isStrictMode,
    concurrentUpdatesByDefaultOverride,
    identifierPrefix,
    onUncaughtError,
    onCaughtError,
    onRecoverableError,
    onDefaultTransitionIndicator,
    transitionCallbacks,
  );

  const rootContainerElement = container;
  listenToAllSupportedEvents(rootContainerElement);  // 在根节点监听所有事件

  return new ReactDOMRoot(root);
}
```

root 创建以及属性

```javascript
 function FiberRootNode(
    containerInfo,
    tag,
    hydrate,
    identifierPrefix,
    onRecoverableError
  ) {
    this.tag = tag; // leagcyRoot/concurrentRoot 
    this.containerInfo = containerInfo; // root根节点
    this.pendingChildren = null;
    this.current = null; // 当前视图的Fiber树
    this.pingCache = null; // 
    this.finishedWork = null; // 被提交的fiberTree
    this.timeoutHandle = noTimeout; // 定时器任务（宏任务）
    this.context = null; // 上下文
    this.pendingContext = null; // getContextForSubtree(parentComponent) 的值
    this.callbackNode = null; // scheduler 体系的 Task 类型
    this.callbackPriority = NoLane; // scheduler 体系的 优先级 1、2、3、4、5
    this.eventTimes = createLaneMap(NoLanes); 
    // 长度32位的数据 number[]32 , 某个事件触发的时间点 markRootUpdated 在finished的时候再清除。
    this.expirationTimes = createLaneMap(NoTimestamp); 
    // 长度32位的数据 number[]32 , 某个事件触发的过期时间 markStarvedLanesAsExpired 
    // 在finished的时候再清除，根据lane 和 eventTime 算出来的。

    // 优先级调度相关
    this.pendingLanes = NoLanes;
    this.suspendedLanes = NoLanes;
    this.pingedLanes = NoLanes;
    this.expiredLanes = NoLanes;
    this.mutableReadLanes = NoLanes;
    this.finishedLanes = NoLanes; // commit  render 
    this.entangledLanes = NoLanes;
    this.entanglements = createLaneMap(NoLanes);
    // 就是一个标识字符，一般为空
    this.identifierPrefix = identifierPrefix;
    this.onRecoverableError = onRecoverableError;

    {
      this.mutableSourceEagerHydrationData = null;
    }

    {
      this.effectDuration = 0;
      this.passiveEffectDuration = 0;
    }

    {
      this.memoizedUpdaters = new Set();
      var pendingUpdatersLaneMap = (this.pendingUpdatersLaneMap = []);
      // 只有在 addFiberToLanesMap  movePendingFibersToMemoized 才会用到，
      for (var _i = 0; _i < TotalLanes; _i++) {
        pendingUpdatersLaneMap.push(new Set());
      }
    }
    ...
  }

```

hostRootFiber 创建

```javascript
 function FiberNode(tag, pendingProps, key, mode) {
    this.tag = tag; // 其实就是fiber类型 26种
    this.key = key; // 开发者指定的key，可以为null
    this.elementType = null;
    this.type = null;
    this.stateNode = null; // Fiber
    // 结构属性
    this.return = null;
    this.child = null;
    this.sibling = null;
    this.index = 0;
    this.ref = null;
    // 状态属性
    this.pendingProps = pendingProps;
    this.memoizedProps = null;
    this.updateQueue = null;
    this.memoizedState = null;
    this.dependencies = null;
    this.mode = mode; // Effects
    // 优先级 副作用
    this.flags = NoFlags;
    this.subtreeFlags = NoFlags;
    this.deletions = null;
    this.lanes = NoLanes;
    this.childLanes = NoLanes;
    this.alternate = null;
    ...
  } 

```

### render流程
<font style="color:rgb(89, 89, 89);">前期准备完成后，开始 render 的过程。就是reconciler和Scheduler的过程，首先获取当前更新优先级，创建更新对象，将更新对象加入UpdateQueue中，</font>

<font style="color:rgb(89, 89, 89);">然后开启调度，调度会在下个宏任务的时候执行workLoop。</font>

<font style="color:rgb(89, 89, 89);">workLoop过程就是递归执行每个节点的beginWork，completeWork的过程，beginWork会给每个节点创建workInProgress的fiber，这是向下递的过程。</font>

<font style="color:rgb(89, 89, 89);">completeWork会从下向上依次生成Dom树，并会打上副作用标记，直到根节点，这是归的过程。</font>

<font style="color:rgb(89, 89, 89);">然后进入commit阶段，commit会经历三个阶段</font>

1. <font style="color:rgb(89, 89, 89);">beforeMutation</font>
2. <font style="color:rgb(89, 89, 89);">mutation</font>
3. <font style="color:rgb(89, 89, 89);">layout</font>



![画板](https://cdn.nlark.com/yuque/0/2026/jpeg/462392/1768267988701-c08ebd7a-b516-47df-8ca4-156e9a8d3168.jpeg)



![画板](https://cdn.nlark.com/yuque/0/2026/jpeg/462392/1768287759093-ba9d2cd4-6861-4789-9f33-8a24b97eb40d.jpeg)



### beginWork
beginWork函数就是根据不同的节点类型（如函数组件、类组件、html 标签、树的根节点等），调用不同的函数，来得到下一个将要处理的 fiber 节点。后续再通过这个新的 fiber 节点，递归后续的 jsx，直到全部遍历完



![画板](https://cdn.nlark.com/yuque/0/2026/jpeg/462392/1768962108605-c36ee4bc-6d78-437a-9378-8bd17c5af688.jpeg)

```typescript
function beginWork(current,workInProgress, renderLanes,) {

  if (current !== null) {
;
  if (
    !hasScheduledUpdateOrContext &&
    (workInProgress.flags & DidCapture) === NoFlags
  ) {
    didReceiveUpdate = false;
    return attemptEarlyBailoutIfNoScheduledUpdate(
      current,
      workInProgress,
      renderLanes,
    );
  }


  workInProgress.lanes = NoLanes;
  switch (workInProgress.tag) { 
    case LazyComponent: 
    case FunctionComponent: {
      const Component = workInProgress.type;
      const unresolvedProps = workInProgress.pendingProps;
      const resolvedProps =  workInProgress.elementType === Component ? unresolvedProps : resolveDefaultProps(Component, unresolvedProps);
      return updateFunctionComponent(
        current,
        workInProgress,
        Component,
        resolvedProps,
        renderLanes,
      );
    }
    case ClassComponent: {
      const Component = workInProgress.type;
      const unresolvedProps = workInProgress.pendingProps;
      return updateClassComponent(
        current,
        workInProgress,
        Component,
        resolvedProps,
        renderLanes,
      );
    }
    case HostRoot:  // 根root节点
      return updateHostRoot(current, workInProgress, renderLanes);
    case HostComponent: // 原生标签div，p
      return updateHostComponent(current, workInProgress, renderLanes);
    case HostText:
      return updateHostText(current, workInProgress);
  }

}
```

### completeWork
<font style="color:rgb(102, 102, 102);">completeWork主要的作用有这么几点：</font><font style="color:rgb(89, 89, 89);">创建DOM，打标签，收集子标签，构建离屏DOM树</font>

![画板](https://cdn.nlark.com/yuque/0/2026/jpeg/462392/1768980787903-d679b8a4-f480-4bba-ab91-b3ee488ab6c6.jpeg)



```typescript
function completeWork(
  current: Fiber | null,
  workInProgress: Fiber,
  renderLanes: Lanes,
): Fiber | null {
  const newProps = workInProgress.pendingProps; // 待处理的新属性

  popTreeContext(workInProgress);
  switch (workInProgress.tag) {
    case IndeterminateComponent:
    case LazyComponent:
    case SimpleMemoComponent:
    case FunctionComponent:
    case ForwardRef:
    case Fragment:
    case Mode:
    case Profiler:
    case ContextConsumer:
    case MemoComponent:
      bubbleProperties(workInProgress);
      return null;
    case ClassComponent: {
      bubbleProperties(workInProgress);
      return null;
    }
   
    case HostComponent: {
      const rootContainerInstance = getRootHostContainer();
      const type = workInProgress.type; // 原生标签名
      if (current !== null && workInProgress.stateNode != null) {
        updateHostComponent(     // 处理原生属性更新
          current,
          workInProgress,
          type,
          newProps,
          rootContainerInstance,
        );

        if (current.ref !== workInProgress.ref) {   // 处理ref
          markRef(workInProgress);
        }
      } else {

          const instance = createInstance(      // 创建原生节点
            type,
            newProps,
            rootContainerInstance,
            currentHostContext,
            workInProgress,
          );
          appendAllChildren(instance, workInProgress, false, false); // 子dom加入到当前dom
          workInProgress.stateNode = instance;

          if (
            finalizeInitialChildren(   // 给标签添加属性
              instance,
              type,
              newProps,
              rootContainerInstance,
              currentHostContext,
            )
          ) {
            markUpdate(workInProgress);
          }
        
        if (workInProgress.ref !== null) {
          // If there is a ref on a host node we need to schedule a callback
          markRef(workInProgress);
        }
      }
      bubbleProperties(workInProgress);
      return null;
    }
  }
}
```



### update类型
初始阶段，类组件的setState，forceUpdate 会去创建Update对象

函数组件的 Update，UpdateQueue 略有不同。



```typescript
export type Update<State> = {
  eventTime: number,
  lane: Lane, // 优先级

  tag: 0 | 1 | 2 | 3, // 更新类型
  payload: any, // 携带的参数
  callback: (() => mixed) | null,  // setState的回调

  next: Update<State> | null, // 下一个
};

export type SharedQueue<State> = {|
  pending: Update<State> | null,  // 新收集的，单向循环链表
  lanes: Lanes,  // 所有的update的合集
|};

export type UpdateQueue<State> = {
  baseState: State, // 老的，基本状态，子节点
  firstBaseUpdate: Update<State> | null, // 第一个，单向链表
  lastBaseUpdate: Update<State> | null,
  shared: SharedQueue<State>, // 新的 queue
  effects: Array<Update<State>> | null,
|};

export const UpdateState = 0;  // 初次渲染
export const ReplaceState = 1;
export const ForceUpdate = 2;
export const CaptureUpdate = 3;
```







































  


触发条件

setState，useState，forceUpdate



![画板](https://cdn.nlark.com/yuque/0/2026/jpeg/462392/1770172535009-ce0384d8-3fdc-4ff0-b6aa-49c596ac92be.jpeg)

整体流程：

1. 调用dispatcher，触发更新
2. 获取优先级，创建更新对象，加入到更新队列
3. 开启调度，合并任务，scheduler开始调度
4. 在下一次事件循环中开始renderRoot，workLoop，commit过程



useState调用会返回dispatcher方法来更新state

```javascript
function dispatchSetState<S, A>(
  fiber: Fiber,
  queue: UpdateQueue<S, A>,
  action: A,
) {
  // 获取 currentUpdatePriority全局变量 更新优先级 
  const lane = requestUpdateLane(fiber);
  // 创建更新对象
  const update: Update<S, A> = {
    lane,
    action,
    hasEagerState: false,
    eagerState: null,
    next: (null: any),
  };
  // 判断 alternate 属性， 判断是否是初始化流程
  if (isRenderPhaseUpdate(fiber)) {
    enqueueRenderPhaseUpdate(queue, update);
  } else {   // 更新流程
    const alternate = fiber.alternate;
    if (
      fiber.lanes === NoLanes &&
      (alternate === null || alternate.lanes === NoLanes)
    ) {
      // The queue is currently empty, which means we can eagerly compute the
      // next state before entering the render phase. If the new state is the
      // same as the current state, we may be able to bail out entirely.
      const lastRenderedReducer = queue.lastRenderedReducer;
      if (lastRenderedReducer !== null) {
        let prevDispatcher;
        const currentState: S = (queue.lastRenderedState: any);        // 旧的 state
        const eagerState = lastRenderedReducer(currentState, action);  // 新的 state
  
        update.hasEagerState = true;
        update.eagerState = eagerState;
          // 判断state 是否相等  相等就取消这次更新任务
        if (is(eagerState, currentState)) {  
            enqueueConcurrentHookUpdateAndEagerlyBailout(fiber,queue,update,lane);
            return;
          }
      }
    }
    
   // 更新对象加入队列， queue.interleaved = update update是一个环形链表，并标记本节点优先级，父节点优先级
    const root = enqueueConcurrentHookUpdate(fiber, queue, update, lane);
    if (root !== null) {
      const eventTime = requestEventTime();
      scheduleUpdateOnFiber(root, fiber, lane, eventTime); // 开启调度
      entangleTransitionUpdate(root, queue, lane);
    }
  }
}
```

调度开始scheduleUpdateOnFiber中调用 ensureRootIsScheduled 这个函数中 会把更新任务加入到scheduler中调度，但是连续调用多次setState，虽然会生成多个更新，但是在这里会判断更新优先级，合并成一个调度任务。这就是**批量更新**

当开始调度前，如果有调度任务，会取消当前任务，重新生成任务

```javascript
function ensureRootIsScheduled(root: FiberRoot, currentTime: number) {
  const existingCallbackNode = root.callbackNode;
  markStarvedLanesAsExpired(root, currentTime);


  // 获取最高优先级
  const newCallbackPriority = getHighestPriorityLane(nextLanes); 
  // 获取当前回调优先级
  const existingCallbackPriority = root.callbackPriority;  
  if (existingCallbackPriority === newCallbackPriority ) {
    return;
  }

  // 有任务在调度，取消当前任务，创建新的调度任务
  if (existingCallbackNode != null) {  
    cancelCallback(existingCallbackNode);
  }

  let newCallbackNode; // 新调度任务
  // 如果新的任务是同步优先级，比如用户操作，点击事件都是同步优先级
  if (newCallbackPriority === SyncLane) {
    // 将 performSyncWorkOnRoot  加入到 syncQueue
    scheduleSyncCallback(performSyncWorkOnRoot.bind(null, root));      
   // 判断环境是否支持微任务，支持就用微任务调度，不支持就用宏任务
    if (supportsMicrotasks) {
        scheduleMicrotask(() => { flushSyncCallbacks(); });
    } else {
      scheduleCallback(ImmediateSchedulerPriority, flushSyncCallbacks);
    }
    newCallbackNode = null;
  } else {
    // 其他情况会根据当前事件优先级，生成调度优先级，再调度
    let schedulerPriorityLevel;
    switch (lanesToEventPriority(nextLanes)) {
      case DiscreteEventPriority:
        schedulerPriorityLevel = ImmediateSchedulerPriority;
        break;
      case ContinuousEventPriority:
        schedulerPriorityLevel = UserBlockingSchedulerPriority;
        break;
      case DefaultEventPriority:
        schedulerPriorityLevel = NormalSchedulerPriority;
        break;
      case IdleEventPriority:
        schedulerPriorityLevel = IdleSchedulerPriority;
        break;
      default:
        schedulerPriorityLevel = NormalSchedulerPriority;
        break;
    }
    newCallbackNode = scheduleCallback(
      schedulerPriorityLevel,
      performConcurrentWorkOnRoot.bind(null, root),
    );
  }

  root.callbackPriority = newCallbackPriority;
  root.callbackNode = newCallbackNode;
}
```

在class组件中调用setState方法时，会调用原型链上绑定的 enqueueSetState 方法

```javascript
const classComponentUpdater = {
  isMounted,
  enqueueSetState(inst, payload, callback) {
    const fiber = getInstance(inst); // 当前class组件的fiber
    const eventTime = requestEventTime();
    const lane = requestUpdateLane(fiber);// 更新优先级

    const update = createUpdate(eventTime, lane);  //  创建一个更新对象
    update.payload = payload;
    if (callback !== undefined && callback !== null) {
      update.callback = callback;
    }

    const root = enqueueUpdate(fiber, update, lane);   // 将更新对象插入到更新队列中
    if (root !== null) {
      scheduleUpdateOnFiber(root, fiber, lane, eventTime);   // 调度更新
      entangleTransitions(root, fiber, lane);
    }

    if (enableSchedulingProfiler) {
      markStateUpdateScheduled(fiber, lane);
    }
  },
  enqueueReplaceState(inst, payload, callback) {},
  enqueueForceUpdate(inst, callback) {},
};

```

接下来beginWork调用updateClassComponent函数，处理state的时候调用getStateFromUpdate来更新state，并且会绑定到fiber.memoizedState

```javascript
function getStateFromUpdate<State>(
  workInProgress: Fiber,
  queue: UpdateQueue<State>,
  update: Update<State>,
  prevState: State,
  nextProps: any,
  instance: any,
): any {
  switch (update.tag) {
    case CaptureUpdate: {
      workInProgress.flags =(workInProgress.flags & ~ShouldCapture) | DidCapture;
    }
    // 更新state 
    case UpdateState: {
      const payload = update.payload;
      let partialState;
      if (typeof payload === 'function') {
        partialState = payload.call(instance, prevState, nextProps);
      } else {
        partialState = payload;
      }
      if (partialState === null || partialState === undefined) {
        return prevState;
      }
      // 将新的 state 和旧的 state 进行合并
      return assign({}, prevState, partialState);
    }
    case ForceUpdate: {
      hasForceUpdate = true;
      return prevState;
    }
  }
  
  return prevState;
}
```


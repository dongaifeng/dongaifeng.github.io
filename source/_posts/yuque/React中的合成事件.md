### 事件执行顺序
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/462392/1769507010904-acfb0258-1b05-486f-a62f-da8771fd1971.png)

### 初始化根节点监听所有事件
![画板](https://cdn.nlark.com/yuque/0/2026/jpeg/462392/1769154557564-6f945b0b-a71f-464e-b3e9-d3f9b3daa2d7.jpeg)

 

### 点击触发事件
![画板](https://cdn.nlark.com/yuque/0/2026/jpeg/462392/1769504288001-8ccdff1c-f53b-4f47-84e3-fcb829e9569c.jpeg)



点击事件，开始收集事件回调函数

```typescript
function extractEvents(
  dispatchQueue: DispatchQueue,
  domEventName: DOMEventName, //  事件名
  targetInst: null | Fiber,		// 事件目标fiber
  nativeEvent: AnyNativeEvent, // 原生事件对象
  nativeEventTarget: null | EventTarget, // 事件目标target
  eventSystemFlags: EventSystemFlags, // 捕获冒泡标志 捕获4，冒泡0
  targetContainer: EventTarget, // root
): void {
  const reactName = topLevelEventsToReactNames.get(domEventName);
 
  let SyntheticEventCtor = SyntheticEvent;
  let reactEventType: string = domEventName;
  
  switch (domEventName) { 
    case 'focusin':
      reactEventType = 'focus';
      SyntheticEventCtor = SyntheticFocusEvent;
      break;
    case 'focusout':
      reactEventType = 'blur';
      SyntheticEventCtor = SyntheticFocusEvent;
      break;
    case 'beforeblur':
    case 'afterblur':
      SyntheticEventCtor = SyntheticFocusEvent;
      break;
    case 'click':
      if (nativeEvent.button === 2) {
        return;
      }
      // ...
    default:
      break;
  }

  const inCapturePhase = (eventSystemFlags & IS_CAPTURE_PHASE) !== 0;   //   // 是否是捕获，在18版本中，会经过两次注册，一次是冒泡，一次是捕获，我们只用看冒泡即可。
  if (
    enableCreateEventHandleAPI &&
    eventSystemFlags & IS_EVENT_HANDLE_NON_MANAGED_NODE
  ) {

  } else {

    const accumulateTargetOnly =!inCapturePhase && domEventName === 'scroll';

    // 从目标节点向上遍历，找到每一层fiber的memoizedProps 上的 回调函数, 并存入到 listeners 数组中
    const listeners = accumulateSinglePhaseListeners( 
      targetInst,
      reactName,
      nativeEvent.type,
      inCapturePhase,
      accumulateTargetOnly,
      nativeEvent,
    );
    if (listeners.length > 0) {
      // 合成事件对象，并加入的 dispatchQueue 中
      const event = new SyntheticEventCtor(
        reactName,
        reactEventType,
        null,
        nativeEvent,
        nativeEventTarget,
      );
      dispatchQueue.push({event, listeners});
    }
  }
}
```

依次触发函数执行，模拟捕获，冒泡过程

```typescript
function processDispatchQueueItemsInOrder(
  event: ReactSyntheticEvent,
  dispatchListeners: Array<DispatchListener>,
  inCapturePhase: boolean,
): void {
  let previousInstance;
  // 捕获 从数组最后向前执行
  if (inCapturePhase) {
    for (let i = dispatchListeners.length - 1; i >= 0; i--) {
      const {instance, currentTarget, listener} = dispatchListeners[i];
      if (instance !== previousInstance && event.isPropagationStopped()) {
        return;
      }
      executeDispatch(event, listener, currentTarget);
      previousInstance = instance;
    }
  // 冒泡
  } else {
    for (let i = 0; i < dispatchListeners.length; i++) {
      const {instance, currentTarget, listener} = dispatchListeners[i];
      if (instance !== previousInstance && event.isPropagationStopped()) {
        return;
      }
      executeDispatch(event, listener, currentTarget);
      previousInstance = instance;
    }
  }
}
```


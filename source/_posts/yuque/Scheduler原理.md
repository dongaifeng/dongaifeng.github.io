---
title: Scheduler原理
---


react版本：18.3.1

位置：packages\scheduler\src\forks\Scheduler.js

### 是什么？
对不同优先级任务进行调度，使高优先级先执行，低优先级后执行。

### 解决的问题？
<font style="color:rgb(25, 27, 31);">解决页面卡顿问题，js单线程，大任务会阻塞浏览器的绘制，导致页面卡顿。</font>

### 怎么解决的？
把任务分片，在浏览器空余时间去执行，通过优先级分配顺序

### 怎么使用？
```javascript
const { unstable_scheduleCallback } = require("scheduler")
const tasks = [1,1,2,2,3,3,4,4,1,2,3,4,1,2,3,4,3,2,1,1,1,1,1]

tasks.forEach((priority , i) => {
  unstable_scheduleCallback(priority , ()=>{
    console.log(`优先级${priority}` , `第${i}任务`)
  })
})
console.log("同步任务")
Promise.resolve().then(res => console.log("微任务"))

// 输出：
// 同步任务
// 微任务
// 优先级1，第0个任务
// 优先级1，第1个任务
// 优先级1，第8个任务
// 优先级1，第12个任务
// 优先级1，第18个任务
// 优先级1，第19个任务
// 优先级1，第20个任务
// 优先级1，第21个任务
// 优先级2，第2个任务
// 优先级2，第3个任务
// 优先级2，第9个任务
// 优先级2，第13个任务
//  ...
```

<font style="color:rgb(77, 77, 77);">Scheduler中的优先级</font>

```javascript

export const ImmediatePriority = 1;     // 超时优先级：需要立即执行的任务       
export const UserBlockingPriority = 2;  // 用户阻塞优先级：用户交互相关任务    
export const NormalPriority = 3;        // 普通优先级：常规更新任务         
export const LowPriority = 4;           // 低优先级：可延迟执行的任务         
export const IdlePriority = 5;          // 空闲优先级：在空闲时执行的任务   
```

Scheduler中的全局变量

```javascript
var maxSigned31BitInt = 1073741823;

// 不同优先级对应时间，会和当前时间相加，计算出过期时间点
var IMMEDIATE_PRIORITY_TIMEOUT = -1;

var USER_BLOCKING_PRIORITY_TIMEOUT = 250;
var NORMAL_PRIORITY_TIMEOUT = 5000;
var LOW_PRIORITY_TIMEOUT = 10000;

var IDLE_PRIORITY_TIMEOUT = maxSigned31BitInt;

var taskQueue = [];  // 立即执行任务队列
var timerQueue = [];  // 延迟执行任务队列

var taskIdCounter = 1; // 任务ID计数器

var isSchedulerPaused = false;

var currentTask = null;  // 当前正在执行的任务对象
var currentPriorityLevel = NormalPriority;  // 当前调度器优先级

var isPerformingWork = false;  // 是否正在执行工作循环

var isHostCallbackScheduled = false;  // 是否已安排工作循环调度
var isHostTimeoutScheduled = false;  // 是否已安排延迟任务检查
```

小顶堆

<font style="color:rgb(25, 27, 31);">本质就是一棵二叉树结构，这里用数组模拟的，它的堆顶永远维持着最小值（最高任务），对外暴露3个方法，push 往堆中塞入一个元素，pop 弹出堆顶元素，peek获取堆顶元素</font>

用节点的 sortIndex作为判断依据 ，如果比较不了，就是用ID，也就是顺序了

```javascript
function compare(a, b) {
  const diff = a.sortIndex - b.sortIndex;
  return diff !== 0 ? diff : a.id - b.id;
}
```



Scheduler暴露出的api

```javascript
export {
  ImmediatePriority as unstable_ImmediatePriority,
  UserBlockingPriority as unstable_UserBlockingPriority,
  NormalPriority as unstable_NormalPriority,
  IdlePriority as unstable_IdlePriority,
  LowPriority as unstable_LowPriority,
  unstable_runWithPriority,
  unstable_next,
  unstable_scheduleCallback,  // 入口方法，开启调度
  unstable_cancelCallback,    // 取消任务
  unstable_wrapCallback,			// 包裹任务
  unstable_getCurrentPriorityLevel,
  shouldYieldToHost as unstable_shouldYield,
  unstable_requestPaint,
  unstable_continueExecution,
  unstable_pauseExecution,
  unstable_getFirstCallbackNode,
  getCurrentTime as unstable_now,  // 获取当前时间戳
  forceFrameRate as unstable_forceFrameRate,
};
```



入口函数

根据优先级计算出当前任务的过期时间。

创建当前任务对象

过期时间和当前时间戳比较，延时任务加入到timerQueue，其他任务加入taskQueue

当前任务为延时，开启定期检查timerQueue中任务的定时器

当前任务为task任务，开启任务调度

```javascript
function unstable_scheduleCallback(priorityLevel, callback, options) {
  var currentTime = getCurrentTime(); // 获取当前时间

  var startTime; // 计算任务开始时间，注意delay参数，如果有delay参数，则表示当前任务为延时任务，会加入timerQueue中
  if (typeof options === 'object' && options !== null) {
    var delay = options.delay;
    if (typeof delay === 'number' && delay > 0) {
      startTime = currentTime + delay;
    } else {
      startTime = currentTime;
    }
  } else {
    startTime = currentTime;
  }

  var timeout;
  switch (priorityLevel) { // 根据优先级 计算任务过期时间
    case ImmediatePriority:
      timeout = IMMEDIATE_PRIORITY_TIMEOUT;
      break;
    case UserBlockingPriority:
      timeout = USER_BLOCKING_PRIORITY_TIMEOUT;
      break;
    case IdlePriority:
      timeout = IDLE_PRIORITY_TIMEOUT;
      break;
    case LowPriority:
      timeout = LOW_PRIORITY_TIMEOUT;
      break;
    case NormalPriority:
    default:
      timeout = NORMAL_PRIORITY_TIMEOUT;
      break;
  }

  var expirationTime = startTime + timeout;  // 计算任务过期时间

  var newTask = {  // 创建任务对象
    id: taskIdCounter++, // 任务id，自增
    callback, // 任务本身回调函数
    priorityLevel, // 任务优先级
    startTime,      // 任务开始时间
    expirationTime,   // 任务过期时间
    sortIndex: -1,      // 用于小顶堆排序的索引，会用来计算优先级，会被赋值expirationTime
  };

  if (startTime > currentTime) { // 延时任务，加入timerQueue中
      newTask.sortIndex = startTime;
      push(timerQueue, newTask);

      cancelHostTimeout(); // 取消定期检查timerQueue
      requestHostTimeout(handleTimeout, startTime - currentTime); // 开启定期检查timerQueue中的任务，检查是否到期
    }
  } else {
    newTask.sortIndex = expirationTime;
    push(taskQueue, newTask);  // 加入到 taskQueue
    requestHostCallback(flushWork); // 开启任务调，传入flushWork
  }

  return newTask;
}
```

通过Scheduler调度的任务，都会在下一个宏任务中去执行，schedulePerformWorkUntilDeadline的定义会根据环境情况去实现

依次检查setImmediate，MessageChannel，setTimeout是否可用

```javascript
function requestHostCallback(callback) {
  scheduledHostCallback = callback;  //  注意，这是全局变量，callback为flushWork
  
  // 开启任务调度循环, 此函数会根据环境选择合适的异步调度API，实现宏任务
  schedulePerformWorkUntilDeadline();   
}

```



```javascript
let schedulePerformWorkUntilDeadline;
if (typeof localSetImmediate === 'function') { // setImmediate
  schedulePerformWorkUntilDeadline = () => {
    localSetImmediate(performWorkUntilDeadline);
  };
} else if (typeof MessageChannel !== 'undefined') { // MessageChannel
  const channel = new MessageChannel();
  const port = channel.port2;
  channel.port1.onmessage = performWorkUntilDeadline;
  schedulePerformWorkUntilDeadline = () => {
    port.postMessage(null);
  };
} else {                                            // setTimeout
  schedulePerformWorkUntilDeadline = () => {
    localSetTimeout(performWorkUntilDeadline, 0);
  };
}
```

循环执行taskQueue中的任务

会判断任务池中的任务

```javascript
const performWorkUntilDeadline = () => {
  if (scheduledHostCallback !== null) {  // 判断当前任务是否为空
    const currentTime = getCurrentTime();
    
    //全局的startTime，用来记录当前这批任务的调度开始时间，用来判断是否用完切片用的。
    startTime = currentTime; 
    const hasTimeRemaining = true;
    let hasMoreWork = true;
    try {
      // 执行flushWork
      hasMoreWork = scheduledHostCallback(hasTimeRemaining, currentTime);   
    } finally {
        // 如果task队列中还有，继续在下一个宏任务中调度
      if (hasMoreWork) {
        schedulePerformWorkUntilDeadline(); 
      } else {
        isMessageLoopRunning = false;
        scheduledHostCallback = null;
      }
    }
  } else {
    isMessageLoopRunning = false;
  }
  needsPaint = false;
};
```

```javascript
function flushWork(hasTimeRemaining, initialTime) {
  isHostCallbackScheduled = false; // 重置调度状态，防止重复调度

  // 如果有定时检查timeQueue，取消检查定时器
  if (isHostTimeoutScheduled) {
    isHostTimeoutScheduled = false;
    cancelHostTimeout();           
  }

  isPerformingWork = true; // 标记正在执行工作循环中
  const previousPriorityLevel = currentPriorityLevel; // 保存当前优先级
  try {
      // ...
      return workLoop(hasTimeRemaining, initialTime);
   
  } finally {
    currentTask = null; // 清理当前任务引用 - 当前任务已完成或暂停
    currentPriorityLevel = previousPriorityLevel; // 恢复原始优先级
    isPerformingWork = false; // 标记 工作循环已完成
  }
}
```



```javascript
function workLoop(hasTimeRemaining, initialTime) {
  let currentTime = initialTime;
  advanceTimers(currentTime); // 检查timerQueue中是否有延时任务
  currentTask = peek(taskQueue);  // 取出第一个任务，最高优先级

  
  while (currentTask !== null) {

    // 如果 expriationTime > currentTime 说明任务还没有过期，
    // 过期任务，不会再调用，因为调度是宏任务
    // shouldYieldToHost 判断浏览器是否还有空余时间（5ms），没有就跳出本次循环，下个宏任务执行
    if (
      currentTask.expirationTime > currentTime &&
      (!hasTimeRemaining || shouldYieldToHost())
    ) {
      break;
    }
    
    const callback = currentTask.callback;
    if (typeof callback === 'function') {
      currentTask.callback = null;   // 先置空任务，防止重复调用
      currentPriorityLevel = currentTask.priorityLevel;
      // 判断当前任务是否过期，chuandaocallback中，供用户使用
      const didUserCallbackTimeout = currentTask.expirationTime <= currentTime; 

      // 执行任务，可能有返回的新任务
      const continuationCallback = callback(didUserCallbackTimeout);
      
      // 任务未完成，继续赋值回调函数
      if (typeof continuationCallback === 'function') {
        currentTask.callback = continuationCallback;  
      } 
        
      // 任务完成了，删除任务
      else {
        if (currentTask === peek(taskQueue)) {  
          pop(taskQueue);
        }
      }
      advanceTimers(currentTime);  // 执行完一个任务，就去检查timerQueue中是否有延时任务
    } else {
      pop(taskQueue);
    }
    
    currentTask = peek(taskQueue); // 下次循环
  }
  
 
  // 如果taskQueue还有任务，返回true，表示还有任务未完成, 需要继续调度
  if (currentTask !== null) {  
    return true;
  } else { // taskQueue已空，检查timerQueue中是否有延时任务
    const firstTimer = peek(timerQueue);
    if (firstTimer !== null) {
      requestHostTimeout(handleTimeout, firstTimer.startTime - currentTime); // 开定时器，执行handleTimeout
    }
    return false;
  }
}
```

advanceTimers用来检查timerQueue中是否有到期的延时任务，如果有则转移到taskQueue中

这个函数会在workLoop开始循环前调用，在一个任务执行结束后调用。

并且也会开始定时器，定时调用

```javascript
  // 循环检查timerQueue中是否有到期的延时任务，如果有则转移到taskQueue中
function advanceTimers(currentTime) { 
  
  let timer = peek(timerQueue);
  while (timer !== null) {
    if (timer.callback === null) {
      
      pop(timerQueue);
    } else if (timer.startTime <= currentTime) {  // 检查是否过期了
      
      pop(timerQueue); // 从timerQueue删除当前任务
      timer.sortIndex = timer.expirationTime;
      push(taskQueue, timer); // 加入到 taskQueue
    
    } 
    
    timer = peek(timerQueue);
  }
}
```

检查timerQueue

当没有开启调度，并且taskQueue有任务，就开启调度

当taskQueue没有任务，timerQueue有任务就再开启定时器，检查timerQueue

```javascript
function handleTimeout(currentTime) {
  isHostTimeoutScheduled = false;
  advanceTimers(currentTime);

  if (!isHostCallbackScheduled) {
    if (peek(taskQueue) !== null) {
      isHostCallbackScheduled = true;
      requestHostCallback(flushWork);  // 开启调度
    } else {
      const firstTimer = peek(timerQueue);
      if (firstTimer !== null) {
        requestHostTimeout(handleTimeout, firstTimer.startTime - currentTime); // 延时检查
      }
    }
  }
}
```

shouldYieldToHost用来判断浏览器空余时间是否用完，是否要退出调度

```javascript
var frameInterval = frameYieldMs;
function shouldYieldToHost() {
  var timeElapsed = exports.unstable_now() - startTime;
  if (timeElapsed < frameInterval) {
    return false; // 说明不应该中断，时间片还没有用完
  } 
  return true; // 说明时间片已经用完了
}
```








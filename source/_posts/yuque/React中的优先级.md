### 位运算基础
在React中是通过二进制字符串表示优先级，位运算可以很好的做优先级运算

场景：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/462392/1768189501366-ef6fbc06-7278-4887-a6ad-2ec9559a6192.png)

在这种情况，<font style="color:rgb(77, 77, 77);">我们使用二进制会方便很多 </font>

二进制相关的运算

+ 与 （&）：只要有一位为0，最终就为0
+ 或（ | ）：只要有一位为1，最终就为1
+ 非（~）：就是0 1 互换
+ 异或（^）：如果两个二进制位不相同，那么结果为1，相同为0
+ 左移( << ) ：左移2，就是在二进制后再加2个0

下面我们来看下位运算在权限系统的实际运用：



下载	打印	查看	审核	详情	删除	编辑	创建

0	0	0	0	0	0	0	0

对应位置上的数字：0就算代表没有权限，1代表有权限

0000 0001 表示有创建的权限

0000 0011 表示编辑和创建的权限

**添加权限 ：**使用或运算  |

0000 0011 表示编辑和创建的权限，再加一个打印的权限

0000 0011 | 0100 0011 = 0100 0011

**删除权限：**使用异或 ^

0100 0011 表示打印、编辑和创建的权限，我们要删除打印的权限

0100 0011 ^ 0100 0000 = 0000 0011

**判断是不是有某一个权限：**使用与 &

0100 0011 表示打印、编辑和创建的权限。

0100 0000 表示打印权限。

判断是否有打印的权限

0100 0000 & 0100 0000 = 0100 0000  有打印权限

<font style="color:rgb(77, 77, 77);">判断是否有 下载的 权限</font>

0100 0011 & 1000 0000 = 0000 0000  无下载权限

如果返回权限本身，表示有此权限，返回全是0，表示无此权限

```typescript
// 示例1：正数左移（核心等价于乘以2的n次方）
const num = 5; // 十进制5 → 32位二进制：00000000 00000000 00000000 00000101

const res1 = num << 1;
// 输出：5 << 1 = 10，二进制：1010
const res2 = num << 2;
// 输出：5 << 2 = 20，二进制：10100

```

### lane模型
在在React中优先级都是通过lane模型和tag标记来描述优先级的

react使用31位的位图的形式来表示不同的优先级，意味着有31种优先级，它们分别是：

```javascript
export const NoLane: Lane = /*                          */ 0b0000000000000000000000000000000;
export const SyncLane: Lane = /*                        */ 0b0000000000000000000000000000001; // 1
export const InputContinuousHydrationLane: Lane = /*    */ 0b0000000000000000000000000000010; // 2
export const InputContinuousLane: Lane = /*             */ 0b0000000000000000000000000000100; // 4

export const DefaultHydrationLane: Lane = /*            */ 0b0000000000000000000000000001000; //16
export const DefaultLane: Lane = /*                     */ 0b0000000000000000000000000010000; //32

const TransitionHydrationLane: Lane = /*                */ 0b0000000000000000000000000100000;
const TransitionLanes: Lanes = /*                       */ 0b0000000001111111111111111000000;
const TransitionLane1: Lane = /*                        */ 0b0000000000000000000000001000000;
...
const TransitionLane16: Lane = /*                       */ 0b0000000001000000000000000000000;

const RetryLanes: Lanes = /*                            */ 0b0000111110000000000000000000000;
const RetryLane1: Lane = /*                             */ 0b0000000010000000000000000000000;
...
const RetryLane5: Lane = /*                             */ 0b0000100000000000000000000000000;

export const SomeRetryLane: Lane = RetryLane1;

export const SelectiveHydrationLane: Lane = /*          */ 0b0001000000000000000000000000000;

const NonIdleLanes: Lanes = /*                          */ 0b0001111111111111111111111111111; 

export const IdleHydrationLane: Lane = /*               */ 0b0010000000000000000000000000000;
export const IdleLane: Lane = /*                        */ 0b0100000000000000000000000000000;

export const OffscreenLane: Lane = /*                   */ 0b1000000000000000000000000000000;
```

### React中的优先级
在react中共有四大类优先级：<font style="color:rgb(33, 37, 41);">事件优先级、更新优先级、任务优先级、调度优先级。</font>

+ <font style="color:rgb(33, 37, 41);">事件优先级：按照用户事件的交互紧急程度，划分的优先级</font>
+ <font style="color:rgb(33, 37, 41);">更新优先级：事件导致React产生的更新对象（update）的优先级（update.lane）</font>
+ <font style="color:rgb(33, 37, 41);">任务优先级：产生更新对象之后，React去执行一个更新任务，这个任务所持有的优先级</font>
+ <font style="color:rgb(33, 37, 41);">调度优先级：Scheduler中的优先级</font>

### 事件优先级
<font style="color:rgb(89, 89, 89);">在react中一共有4种</font>**<font style="color:rgb(3, 106, 202);">事件优先级</font>**

```javascript
export const DiscreteEventPriority: EventPriority = SyncLane; // 离散事件
export const ContinuousEventPriority: EventPriority = InputContinuousLane; // 持续触发事件
export const DefaultEventPriority: EventPriority = DefaultLane; // 默认事件
export const IdleEventPriority: EventPriority = IdleLane; // 空闲事件
```

### <font style="color:rgb(33, 37, 41);">更新优先级</font>
<font style="color:rgb(33, 37, 41);">初始化：render -->updateContainer  --> requestUpdateLane</font>

<font style="color:rgb(33, 37, 41);">更新：setState --> dispatcher --> requestUpdateLane</font>

无论是mount阶段，还是更新阶段做的事情大概如下：

1. 获取更新优先级
2. 创建update对象
3. 将update对象加入fiber的更新队列
4. 开始更新的调度

源码中有一个 `requestUpdateLane` 的函数，是用来获得当前的更新优先级的，所有更新优先级并不是一个固定的值，它会依次判断当前的更新属于什么类型。

不是并发模式，同步优先级，不支持并发更新；

属于transition优先级，返回当前的过渡优先级；

存在更新优先级，通过`getCurrentUpdatePriority`获取事件触发时的优先级

最后返回：事件优先级

```javascript
export function requestUpdateLane(fiber: Fiber): Lane {
  const mode = fiber.mode;
  if ((mode & ConcurrentMode) === NoMode) {   // 1. 同步
    return (SyncLane: Lane);
  } 
    
  const isTransition = requestCurrentTransition() !== NoTransition;
  if (isTransition) {                       // 2.transition
    return currentEventTransitionLane;
  }

  const updateLane: Lane = (getCurrentUpdatePriority(): any);
  if (updateLane !== NoLane) {               // 3.当前是什么样的更新
    return updateLane;
  }

  const eventLane: Lane = (getCurrentEventPriority(): any);
  return eventLane;                         // 4.事件
}
```

执行`onClick`的时候，全局的 `currentUpdatePriority` 已经被设置成了对应的事件优先级，等事件执行完毕，再恢复成原来的优先级，因此在某个事件产生的更新如果去获取`更新优先级`的话，在不满足前两个判断的情况下，它必然会获取到对应的这个`事件优先级`。但是有一些更新并不是由事件产生的，可能是由IO等异步操作产生的，它们的执行可能没有对应的事件，这个时候就走第4个判断，`getCurrentEventPriority`

```javascript
function dispatchDiscreteEvent(
    domEventName,
    eventSystemFlags,
    container,
    nativeEvent
  ) {
    var previousPriority = getCurrentUpdatePriority();
    try {
      setCurrentUpdatePriority(DiscreteEventPriority);
      // 这个函数中包含执行onClick，并且是同步执行
      dispatchEvent(domEventName, eventSystemFlags, container, nativeEvent);
    } finally {
      setCurrentUpdatePriority(previousPriority);
    }
  }

```

<font style="color:rgb(89, 89, 89);">因此可以看出：</font>**<font style="color:rgb(3, 106, 202);">事件和更新并非一一对应</font>**<font style="color:rgb(89, 89, 89);">，换句话说，</font>**<font style="color:rgb(3, 106, 202);">在一个事件中有可能产生多种优先级的更新</font>**

### 任务优先级
<font style="color:rgb(33, 37, 41);">任务优先级被用来区分多个更新任务的紧急程度，它由更新优先级计算而来</font>

<font style="color:rgb(33, 37, 41);">假设产生一前一后两个update，它们持有各自的更新优先级，也会被各自的更新任务执行。经过优先级计算，如果后者的任务优先级高于前者的任务优先级，那么会让Scheduler取消前者的任务调度；如果后者的任务优先级等于前者的任务优先级，后者不会导致前者被取消，而是会复用前者的更新任务，将两个同等优先级的更新收敛到一次任务中；如果后者的任务优先级低于前者的任务优先级，同样不会导致前者的任务被取消，而是在前者更新完成后，再次用Scheduler对后者发起一次任务调度。</font>

<font style="color:rgb(33, 37, 41);">这是任务优先级存在的意义，保证高优先级任务及时响应，收敛同等优先级的任务调度。</font>

```javascript
function ensureRootIsScheduled(root: FiberRoot, currentTime: number) {

  ...

  // 获取nextLanes，顺便计算任务优先级
  const nextLanes = getNextLanes(
    root,
    root === workInProgressRoot ? workInProgressRootRenderLanes : NoLanes,
  );

  // 获取上面计算得出的任务优先级
  const newCallbackPriority = returnNextLanesPriority();

  ...

}
```

### 调度优先级
<font style="color:rgb(33, 37, 41);">一旦任务被调度，那么它就会进入Scheduler，在Scheduler中，这个任务会被包装一下，生成一个属于Scheduler自己的task，这个task持有的优先级就是调度优先级。</font>






























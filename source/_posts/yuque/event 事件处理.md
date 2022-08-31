---
title: event 事件处理
urlname: tehiqr
date: '2020-09-08 18:58:37 +0800'
tags: []
categories: []
---

### vue 的事件分四种

- 原生标签的原生 DOM 事件
- 原生标签的自定义事件
- 组件的原生 DOM 事件
- 组件的的自定义事件

### 方式

- $on 注册事件。也可以 @click 最后也是调用 $on.
- $emit 触发事件
- $off 解绑事件

### 形态

#### 标签的 DOM 事件

在 div 标签的 VNode 里 data 属性中，可以看到我们绑定的 DOM 事件。
![image.png](https://cdn.nlark.com/yuque/0/2020/png/462392/1597825569865-216e4505-3c37-41fd-a7ee-502c800ae96f.png#align=left&display=inline&height=117&margin=%5Bobject%20Object%5D&name=image.png&originHeight=117&originWidth=271&size=23251&status=done&style=none&width=271)

#### 标签自定义事件

同样的自定义事件 myevent 在这里也可以看到
![image.png](https://cdn.nlark.com/yuque/0/2020/png/462392/1597825912298-b3d8bcc4-d0a6-4024-baea-042af478cb83.png#align=left&display=inline&height=191&margin=%5Bobject%20Object%5D&name=image.png&originHeight=191&originWidth=320&size=10216&status=done&style=none&width=320)

#### 组件的 DOM 事件

组件的 DOM 事件，需要加上 native 修饰符 例如 @click.native = ""。
在组件的 vm.$vnode (组件的外壳节点) 上可以看绑定的 DOM 事件。
同时的在父组件的 \_Node.children 里也可看到。
![image.png](https://cdn.nlark.com/yuque/0/2020/png/462392/1597826055440-2666c785-3152-4d4a-a677-aa63394b440a.png#align=left&display=inline&height=288&margin=%5Bobject%20Object%5D&name=image.png&originHeight=288&originWidth=528&size=21022&status=done&style=none&width=528)

#### 组件的自定义事件

组件的 \_events 属性是用来存放本实例上注册的自定义事件。
在 $listeners 也可看到 自定义事件。
![image.png](https://cdn.nlark.com/yuque/0/2020/png/462392/1597826782160-2bfaa8e3-4d4d-4af6-834c-c5932a0ad6e0.png#align=left&display=inline&height=87&margin=%5Bobject%20Object%5D&name=image.png&originHeight=87&originWidth=225&size=19851&status=done&style=none&width=225)

### 来龙去脉

#### 原生标签 DOM 事件

template 被解析变成渲染函数。
![image.png](https://cdn.nlark.com/yuque/0/2020/png/462392/1597975739183-5e79f63a-50df-4a60-aa2b-c5d118cb966d.png#align=left&display=inline&height=239&margin=%5Bobject%20Object%5D&name=image.png&originHeight=239&originWidth=360&size=45684&status=done&style=none&width=360)
渲染函数在执行的时候会变成 Vnode。
![image.png](https://cdn.nlark.com/yuque/0/2020/png/462392/1597975820185-9da249fc-b18e-4e03-864f-6f5f73242ec1.png#align=left&display=inline&height=219&margin=%5Bobject%20Object%5D&name=image.png&originHeight=219&originWidth=312&size=28323&status=done&style=none&width=312)
这部分是发生在更新函数执行的时候。updateComponent 被传入组件的 Watcher，mount 的时候 new Watcher 会执行此函数。

```javascript
updateComponent = () => {
  // _render 主要用于生成vnode
  // _update 拿到Vnode 会进行新旧Vnode对比，然后更新dom。
  vm._update(vm._render(), hydrating);
};
```

下面我们从*update 里面开始探索。
\_upload 里面调用**patch**。
\*\** **\_**patch*\*\**  是在 \src\platforms\web\runtime\index.js 文件中挂载到 Vue 的原型上的。
真正的代码在 src\core\vdom\patch.js 中。
在 patch 函数中进行新旧 dom 对比，调用 **createElm** 创建新节点。

```javascript
function createElm(vnode, insertedVnodeQueue, parentElm, refElm) {
  // 创建孩子节点
  // 判断有无data数据 有的话调用 invokeCreateHooks 处理节点属性
  createChildren(vnode, children, insertedVnodeQueue);
  if (isDef(data)) {
    invokeCreateHooks(vnode, insertedVnodeQueue);
  }

  // 把这个节点插入到父节点
  insert(parentElm, vnode.elm, refElm);
}
```

**invokeCreateHooks 用于处理节点属性**

```javascript
function invokeCreateHooks(vnode, insertedVnodeQueue) {
  // 遍历cbs.create 依次执行里面处理节点属性的方法
  for (let i = 0; i < cbs.create.length; ++i) {
    cbs.create[i](emptyNode, vnode);
  }
  i = vnode.data.hook; // Reuse variable
  if (isDef(i)) {
    if (isDef(i.create)) i.create(emptyNode, vnode);
    if (isDef(i.insert)) insertedVnodeQueue.push(vnode);
  }
}
```

cbs 里有什么？ cbs 是一个大包，里面包含着对节点属性的添加，更新，删除等一系列方法。
我们看到了 updateDomListeners
![image.png](https://cdn.nlark.com/yuque/0/2020/png/462392/1597978827616-50586b8d-0fe6-4337-9a54-ecbbe8d81011.png#align=left&display=inline&height=284&margin=%5Bobject%20Object%5D&name=image.png&originHeight=284&originWidth=352&size=18294&status=done&style=none&width=352)
到这里我们看下流程
\_upload---->**_patch_** ------> createElm ----->invokeCreateHooks ------> updateDomListeners

**updateDomListeners** 才是终极大佬。

```javascript
function updateDOMListeners(oldVnode, vnode) {
  // 拿到 本节点 的 新旧 事件对象 on
  const on = vnode.data.on || {};
  const oldOn = oldVnode.data.on || {};
  target = vnode.elm;
  // 规范一下事件回调函数
  normalizeEvents(on);
  // 更新 处理 事件
  updateListeners(on, oldOn, add, remove, createOnceHandler, vnode.context);
  target = undefined;
}
```

**updateListeners** 函数做了新旧事件对比
1、新旧事件相同，替换旧事件
2、新事件不存在旧事件中，绑定新事件
3、旧事件不存在新事件中，解绑旧事件

```javascript
export function updateListeners(on, oldOn, add, remove, createOnceHandler, vm) {
  let name, def, cur, old, event;
  for (name in on) {
    def = cur = on[name];
    old = oldOn[name];
    event = normalizeEvent(name);

    // 如果新事件有 老事件没有 就给节点添加事件 add
    if (isUndef(old)) {
      if (isUndef(cur.fns)) {
        cur = on[name] = createFnInvoker(cur, vm);
      }
      // 处理 .once 标识符
      if (isTrue(event.once)) {
        cur = on[name] = createOnceHandler(event.name, cur, event.capture);
      }
      add(event.name, cur, event.capture, event.passive, event.params);
    }

    // 如果老事件 和 新事件不同 就把新事件函数 给 老事件
    else if (cur !== old) {
      old.fns = cur;
      on[name] = old;
    }
  }
  // 遍历老事件对象 如果在新事件对象里没有 就删除此事件
  for (name in oldOn) {
    if (isUndef(on[name])) {
      event = normalizeEvent(name);
      remove(event.name, oldOn[name], event.capture);
    }
  }
}
```

**add** 方法 就是 addEventListener
**remove **就是 removeEventListener

```javascript
function add() {
  target.addEventListener(
    name,
    handler,
    supportsPassive ? { capture, passive } : capture
  );
}
```

#### 原生标签自定义事件

原生标签编译成 Vnode 后自定义事件和原生事件存放在同一地方。也会调用 addEventListener。

#### 组件自定义事件

父组件在生成 Vnode 的时候，遇到子组件是组件，并不会初始化子组件，而是生成**组件外壳 vnode**。
父组件在\_update 的时候会调用 createElm。遍历到子组件的时候，发现子组件是个组件。会调用子组件 data.hook.init 初始化子组件。
子组件初始化会生成子组件实例，并调用 $mount。也会调用initEvent。会把从父组件得到的监听事件$listener 遍历放到\_event。
在子组件的调用$emit。会去\_event 里面找到对应的同事件名的函数调用。
以上的流程会在 component 里介绍到。
**initEvents 初始化父组件监听的事件**

```javascript
export function initEvents(vm: Component) {
  vm._events = Object.create(null);

  // 拿到父组件绑定的事件函数，调用updateComponentListeners
  const listeners = vm.$options._parentListeners;
  updateComponentListeners(vm, listeners);
}
```

**updateComponentListeners**

```javascript
export function updateComponentListeners(vm, listeners, oldListeners) {
  target = vm;
  updateListeners(
    listeners,
    oldListeners || {},
    add,
    remove,
    createOnceHandler,
    vm
  );
  target = undefined;
}
```

**updateListeners**
最终也是调用 updateListeners 函数 只是传入的 add，remove 方法不同。

```javascript
function add(event, fn) {
  target.$on(event, fn);
}

// $on 是监听事件 其实就是把父组件的事件加入到子组件的_event里面
Vue.prototype.$on = function (event, fn) {
  const vm = this;
  // 递归调用
  if (Array.isArray(event)) {
    for (let i = 0, l = event.length; i < l; i++) {
      vm.$on(event[i], fn);
    }
  } else {
    // 遍历event 把回调函数fn push到 vm._events[event]
    (vm._events[event] || (vm._events[event] = [])).push(fn);
  }
  return vm;
};
```

**$emit**
$emit 会把事件回调函数从\_event 里拿出来，用 apply，call 调用

```javascript
Vue.prototype.$emit = function (event) {
  const vm = this;

  let cbs = vm._events[event];
  if (cbs) {
    cbs = cbs.length > 1 ? toArray(cbs) : cbs;
    const args = toArray(arguments, 1);

    for (let i = 0, l = cbs.length; i < l; i++) {
      invokeWithErrorHandling(cbs[i], vm, args, vm, info);
    }
  }
  return vm;
};

// 调用回调函数
export function invokeWithErrorHandling(handler, context, args) {
  let res;
  try {
    res = args ? handler.apply(context, args) : handler.call(context);
  } catch (e) {
    handleError(e, vm, info);
  }
  return res;
}
```

#### 组件的 DOM 事件

在创建组件外壳 vnode 的时候，会使用\_c
\_c 里面调用 createComponent，可以看到一行代码
data.on = data.nativeOn
剩下的流程，就和前面的一样了
createElm--->createComponent--->initComponent---->invokeCreateHooks------>updateDOMListeners

### 总结

1. 原生 dom 事件最后都是在 vnode.elm 上用 addEventListener 监听。
1. 自定事件 保存在\_event 里面，当调用 emit 时，回去\_event 找对应的事件执行，事件可能是一个数组，遍历调用。
1. patch 里面的 createComponent 调用 创建组件里面的实例。
1. createElement 里面的 createComponent 是创建组件的外壳 vnode。

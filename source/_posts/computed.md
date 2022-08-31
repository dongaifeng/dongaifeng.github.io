---
title: computed
urlname: wlss15
date: 2020-07-22 13:34:54 +0800
tags: []
categories: []
---

## 整理流程

#### 官网文档说明

计算属性是基于它们的响应式依赖进行**缓存**的。只在相关响应式依赖发生改变时它们才会重新求值。
这就意味着只要   依赖项还没有发生改变，多次访问该计算属性会立即返回之前的计算结果，而**不必再次执行函数**

```javascript
computed: {
  fullName: {
    // getter
    get: function () {
      return 'a'
    },
    // setter
    set: function (newValue) {

    }
  }
}
```

#### computed 是响应式的

computed 设置的 get 和 set 函数，最终会设置到 Object.defineProperty 的 get，set 函数里。这和 vue 的响应式原理是一样的，当读取 computed 时会执行 get。，赋值的时候会执行 set 函数。

#### computed 会被缓存

computed 根据他的依赖项进行计算(根据你设置的 get)，会把值保存到一个变量中，当在 template 中使用计算属性时，会直接返回这个变量。
当依赖项变化，computed 会重新计算，更新这个变量。
computed 控制缓存是根据一个 watcher 的一个属性 **dirty（是否脏了）。**
当 dirty = true 时，读取 computed 会重新计算。
当 dirty = false 时，读取 computed 会使用缓存。

#### 过程

1. 开始每个 computed 会新建自己的 watcher，并设置 watcher.dirty = true, 但是并不会马上计算。
1. 当页面中使用了 computed，会触发 get 函数，触发 watcher.evaluate()去计算。
1. computed  计算完成之后，会设置 watcher.dirty = false
1. 当依赖变化，通知 computed 的 watcher，并设置 watcher.dirty = true

当 computed 依赖了 data 里的数据，当 data 改变。第一步会通知 computed 的 watcher 更新，其实只会重置 dirty = true，并不会计算。第二步会通知模板的 watcher 进行更新渲染，进而重新读取 computed，然后 computed 才会重新计算。

#### 依赖收集

比如 computed 使用了 data 的属性 D，data 里的属性 D 的依赖收集器会同时收集到 computed 的  watcher  和   页模板的  watcher。
template 在读取 computed 的时候，会趁机把 template 的 watcher 推荐给 data，data 就会收集到 template 的 watcher。

## 源码

源码部分我们从一开始 new Vue 的时候看。
流程：initMixin --> \_init --> initState --> initComputed

```javascript
function Vue(options) {
  if (process.env.NODE_ENV !== "production" && !(this instanceof Vue)) {
    warn("Vue is a constructor and should be called with the `new` keyword");
  }
  this._init(options); // _init方法是在 initMixin 中加入的
}

// 初始化init
initMixin(Vue);
```

```javascript
export function initMixin(Vue: Class<Component>) {
  //  _init 方法
  Vue.prototype._init = function (options?: Object) {
    // 构造函数初始化
    vm._self = vm;
    initLifecycle(vm); // $parent,$root,$children,$refs
    initEvents(vm); // 处理父组件传递的监听器
    initRender(vm); // $slots,$scopedSlots,_c,$createElement
    callHook(vm, "beforeCreate");
    initInjections(vm); // resolve injections before data/props
    initState(vm); // 初始化props，methods，data，computed，watch 这里面又是一堆init
    initProvide(vm); // resolve provide after data/props
    callHook(vm, "created");
  };
}
```

```javascript
export function initState(vm: Component) {
  if (opts.computed) initComputed(vm, opts.computed);
}

function initComputed(vm: Component, computed: Object) {
  // 在vm实例上保存computedWatchers
  const watchers = (vm._computedWatchers = Object.create(null));

  for (const key in computed) {
    const userDef = computed[key];
    const getter = typeof userDef === "function" ? userDef : userDef.get;

    //!!!c 给每个计算属性生成一个watcher(注意key),
    // 在Watcher构造函数中会根据 lazy 生成computed_watcher
    watchers[key] = new Watcher(
      vm,
      getter || noop,
      noop,
      computedWatcherOptions // 在上面会声明 const computedWatcherOptions = { lazy: true }
    );

    // 定义计算属性的 get set
    // 在vm实例上可直接访问
    if (!(key in vm)) {
      defineComputed(vm, key, userDef);
    }
  }
}
```

initComputed  这段代码做了几件事
1、每个  computed  配发 watcher
2、defineComputed  定义 cumputed 响应式。
3、收集所有  computed  的  watcher，保存到 vm.\_computedWatchers。

```javascript
export default class Watcher {
  constructor(
    vm: Component,
    expOrFn: string | Function,
    cb: Function,
    options?: ?Object,
    isRenderWatcher?: boolean
  ) {
    this.lazy = !!options.lazy;
    this.dirty = this.lazy; // for lazy watchers
    this.getter = expOrFn; // 把计算属性的get 赋值给 watcher的getter
    //!!!c 这里并不会计算值，只有你再读取 computed，再开始计算
    this.value = this.lazy ? undefined : this.get();
  }
}
```

```javascript
export function defineComputed(
  target: any,
  key: string,
  userDef: Object | Function
) {
  sharedPropertyDefinition.get = createComputedGetter(key);
  sharedPropertyDefinition.set = userDef.set;

  Object.defineProperty(target, key, sharedPropertyDefinition);
}

function createComputedGetter(key) {
  return function computedGetter() {
    const watcher = this._computedWatchers && this._computedWatchers[key];
    // 在计算属性被使用时 会触发这个函数 也就是get
    // 通过key 获取到对应的 _computedWatcher
    // 根据 watcher.dirty 决定 value使用的是缓存的还是重新计算
    if (watcher.dirty) {
      // watcher.evaluate 用来重新计算，更新缓存值，并重置 dirty 为false，表示缓存已更新
      // 只有 dirty 为 true 的时候，才会执行 evaluate
      watcher.evaluate();
    }

    //  此时Dep.target就是当前页面的watcher，
    // 让data的属性(computed的依赖项)收集当前页面的watcher，也就是__ob__下面的dep的subs中
    // 这样data的subs中 就有了页面的watcher，computed的watcher
    // 这样data的数据改变 会直接分别通知 页面的watcher，computed的watcher
    if (Dep.target) {
      watcher.depend();
    }
    return watcher.value;
  };
}
```

defineComputed 做了哪些事？

1. 使用  Object.defineProperty  在实例上定义 computed  属性，所以可以直接访问
1. createComputedGetter 包装返回 get  函数

createComputedGetter 的作用：

1. 计算 computed 的值
1. 依赖收集：让 data  的依赖收集器收集到 computed-watcher 和 页面 watcher。这里有点绕：

**evaluate 调用了 get，在 get 里面，**
**先把当前 watcher，也就是 computed-watcher 赋值给 taget。**
**再调用 getter 计算值，这是也就是你设置的 get，get 会用到 data 里的属性，触发 data 里的属性的 get，让 data         的属性收集到 computed-watcher。**
**然后 popTarget 会把 taget 设为上一个 watcher，也就是页面 watcher。**
**然后再进行上面的 watcher.depend()**

```javascript
  evaluate () {
    this.value = this.get()
    this.dirty = false
  }

  //  收集依赖
  get () {
     // 在这里把当前watcher给了 Dep.target
    pushTarget(this)
    value = this.getter.call(vm, vm) // 计算值

    // 从targetStack（watcher缓存栈）末尾删除本watcher
    // 并且设置target 为上一个 watcher
    popTarget()
    this.cleanupDeps()
    return value
  }

 //!!!c watcher.deps存放的是data属性的dep。比如data：{a.b.c}  就会有a的dep  b的dep  c的dep
  depend () {
    let i = this.deps.length
    while (i--) {
      this.deps[i].depend()
    }
  }
```

## 总结

1 页面更新，读取  computed  的时候，Dep.target  会设置为   页面 watcher。
2 computed  被读取，createComputedGetter  包装的函数触发，第一次会进行计算
computed-watcher.evaluted  被调用，进而 computed-watcher.get  被调用，Dep.target  被设置为  computed- watcher，旧值 页面  watcher  被缓存起来。
3 computed  计算会读取  data，此时  data 就收集到 computed-watcher
同时  computed-watcher  也会保存到  data  的依赖收集器  dep（用于下一步）。
computed  计算完毕，释放 Dep.target，并且 Dep.target  恢复上一个 watcher（页面 watcher）
4 手动  watcher.depend， 让  data  再收集一次  Dep.target，于是  data  又收集到   恢复了的页面 watcher
![image.png](https://cdn.nlark.com/yuque/0/2020/png/462392/1595413657995-73bfab32-8822-4232-b05a-9825024193e8.png#align=left&display=inline&height=484&margin=%5Bobject%20Object%5D&name=image.png&originHeight=484&originWidth=342&size=77839&status=done&style=none&width=342)

---
title: props
urlname: lwylxk
date: '2020-09-08 18:58:37 +0800'
tags: []
categories: []
---

props 做为父子组件通信的重要载体，工作原理是什么？
提出问题：
父组件怎么传值给子组件？
父组件数据改变怎么通知子组件？
子组件数据改变会怎样？

带着问题我们来进行 props 的探索之旅

### 组件初始化

组件初始化，调用 \_int ---->initState -----> initProps

```javascript
function initProps (vm, propsOptions) {
  const propsData = vm.$options.propsData || {}
  const props = vm._props = {}

  const keys = vm.$options._propKeys = []
  const isRoot = !vm.$parent

  for (const key in propsOptions) {
    keys.push(key)
    // 校验props的值
    const value = validateProp(key, propsOptions, propsData, vm)
    // 定义 _props 的属性响应式
    defineReactive(props, key, value)
    }

  	// 代理到Vue实例this上
    if (!(key in vm)) {
      proxy(vm, `_props`, key)
    }
  }
}
```

initProps 遍历 vm.$options.propsData 上的属性，依次给每个属性做响应式，也就是 defineReactive。
defineReactive 在依赖收集的章节有讲，主要是设置\_props 属性的 get，set。跟 data 一样。

propsData 是什么时候挂在的呢？

### propsData

#### 父组件生成 vnode

\_init ----> $mount ----> \_render ----> \_createElement ----> createComponent ----> extractPropsFromVNodeData
父组件 init，生成 render 函数，可以看到子组件的 data 的 attrs 里面存放父组件传来的数据。

```javascript
(function anonymous() {
  with (this) {
    return _c(
      "div",
      { attrs: { id: "app" } },
      [
        _c("comp", { attrs: { hello: "aaa", myprop: msg }, on: { eve: aaa } }, [
          _v("\n        组件的slot\n      "),
        ]),
      ],
      1
    );
  }
});
```

render 函数执行时，遇到 children 是组件，调用 createComponent 去创建子组件的 vnode。在其中会检出 propsData。

```javascript
// 把 props 从data里面检出来，生成vnode的时候会挂在到 componentOptions 上
const propsData = extractPropsFromVNodeData(data, Ctor, tag);

export function extractPropsFromVNodeData(data, Ctor, tag) {
  // 从子组件选项中 拿出props，然后父组件传来的数据比对，检出props。
  const propOptions = Ctor.options.props;
  const res = {};
  const { attrs, props } = data;

  for (const key in propOptions) {
    const altKey = hyphenate(key);

    const keyInLowerCase = key.toLowerCase();

    // 检出props，赋值给res
    checkProp(res, props, key, altKey, true) ||
      checkProp(res, attrs, key, altKey, false);
  }
  return res;
}
```

生成的 propsData 在 Vnode 里面会存放到 this.componentOptions。

#### 子组件初始化

接着上面的流程
vnode ----> \_ \_patch\_\_ ----> createElm ----> createComponent ----> componentVNodeHooks.init ----> 子组件\_init ----> initInternalComponent
父组件 vnode 完成后，开始生成节点。遍历到子节点时组件时，调用 createComponent，接着开始子组件的初始化
在初始化选项的时候

```javascript
export function initInternalComponent(
  vm: Component,
  options: InternalComponentOptions
) {
  const opts = (vm.$options = Object.create(vm.constructor.options));
  const parentVnode = options._parentVnode;
  opts.parent = options.parent;
  opts._parentVnode = parentVnode;

  const vnodeComponentOptions = parentVnode.componentOptions;
  // 将外壳vnode上的 componentOptions 赋值给 vm.$options.propsData
  opts.propsData = vnodeComponentOptions.propsData;
  opts._parentListeners = vnodeComponentOptions.listeners;
  opts._renderChildren = vnodeComponentOptions.children;
  opts._componentTag = vnodeComponentOptions.tag;

  if (options.render) {
    opts.render = options.render;
    opts.staticRenderFns = options.staticRenderFns;
  }
}
```

到这里我们追踪到 vm.$options.propsData 是怎么得到父组件传来的 props 的。

### 父组件改变数据

那么父组件数据改变后会怎样？
data 改变 ----> 触发属性的 set ----> dep.notify ----> watcher.update ----> queueWatcher
queueWatcher 会将 当前 watcher 加入 queue 队列， 用函数 flushSchedulerQueue 包装以后传给 nextTick。

queueWatcher ----> nextTick ----> timerFunc()
nextTick 会将传入的函数加入异步任务队列 callbacks。并触发 timerFunc 函数。将异步任务 加入到微任务队列。

当同步任务执行完成后，开始异步任务。

flushCallbacks ----> flushSchedulerQueue ----> watcher.run ----> watcher.get ----> getter ----> updateComponent
异步任务执行，watcher 开始更新程序，最终调用 updateComponent，开始组件更新。

_render ----> \_update ----> _ \_patch\_\_  ----> patchVnode ----> updateChildren
父组件重新生成 vnode，触发新旧 vnode 对比。

```javascript
function patchVnode(oldVnode, vnode) {
  // 组件在这里开始处理从父组件获取的数据，更新props
  let i;
  const data = vnode.data;
  if (isDef(data) && isDef((i = data.hook)) && isDef((i = i.prepatch))) {
    i(oldVnode, vnode);
  }
}
```

prepatch ----> updateChildComponent
prepatch 是组件在最初\_createElement 的时候挂在在 vnode.data 上的 hooks，作用就是更新组件从父组件传来的数据。

```javascript
export function updateChildComponent(
  vm,
  propsData,
  listeners,
  parentVnode,
  renderChildren
) {
  // !!!props 更新子组件props
  if (propsData && vm.$options.props) {
    const props = vm._props;
    const propKeys = vm.$options._propKeys || [];

    // 遍历 新props 依次更新到组件的 _props
    for (let i = 0; i < propKeys.length; i++) {
      const key = propKeys[i];
      const propOptions = vm.$options.props;
      // 会验证props 值的合法性
      props[key] = validateProp(key, propOptions, propsData, vm);
    }
    vm.$options.propsData = propsData;
  }
}
```

\_props 在上边已经提到会变成响应式，和组件的 data 一样，所以会触发 set ，更新组件。

### 子组件改变 props

官网有提示 不建议在子组件修改 props，但是强行修改的话，因为\_props 是响应式的，所以子组件会更新，父组件不会。
![image.png](https://cdn.nlark.com/yuque/0/2020/png/462392/1599039655106-663df5f9-e177-41a8-a96c-e1d55194605a.png#align=left&display=inline&height=146&margin=%5Bobject%20Object%5D&name=image.png&originHeight=146&originWidth=710&size=24930&status=done&style=shadow&width=710)
**but 如果父组件传的是一个对象，因为这个对象在父组件使用后也会变成响应式，会收集到父组件的 watcher。**
**所以你在子组件修改这个对象里的属性时，父组件也会跟着更新。但是，如果对象被整个替换了，而不是修改内部，那父组件不会更新。**

### 总结

- 子组件初始化 initProps 拿到父组件数据，并设置响应式，并 proxy 代理。
- 父组件数据改变，触发子组件更新，重新获取 props，再触发子组件更新。
- 子组件改变数据，分两种情况，1 基本类型，只之组件更新。2 引用类型，父组件，子组件都会更新。

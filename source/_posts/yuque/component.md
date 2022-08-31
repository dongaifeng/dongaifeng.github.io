---
title: component
urlname: vq11m6
date: 2020-08-24 20:47:39 +0800
tags: []
categories: []
---

## 要走通 component 流程，需要先弄清楚几个概念。

#### 两种 Vnode

组件内 template 生成的 vnode
![image.png](https://cdn.nlark.com/yuque/0/2020/png/462392/1598513688164-19f57ef7-39a0-4843-b4ab-0c9baee68891.png#align=left&display=inline&height=366&margin=%5Bobject%20Object%5D&name=image.png&originHeight=437&originWidth=813&size=32849&status=done&style=shadow&width=681)

- elm 生成的真实节点 真实 DOM
- context 组件上下文对象，组件 template 里的 this 指向，就是组件的实例对象。
- parent 指向组件的外壳 vnode。

组件外壳 vnode
![image.png](https://cdn.nlark.com/yuque/0/2020/png/462392/1598513587351-16b5a2e4-4761-475b-948e-b86350be5472.png#align=left&display=inline&height=344&margin=%5Bobject%20Object%5D&name=image.png&originHeight=441&originWidth=875&size=37516&status=done&style=shadow&width=683)

- componentInstance 组件生成的实例，在这里指向上面的图。
- data 保存组件上的 class style 事件。有 attrs，staticClass，staticStyle

nativeOn 保存原生事件，后面会赋值给 on
hook 包括 init destory prepatch insert 等方法。组件初始化，更新，销毁的钩子函数。

- componentOptions 保存了父组件传给子组件的数据。

Ctor 子组件的构造函数
children 使用组件标签里的内容，就是 slot。
listeners 组件监听的事件。
propsData 组件的 props。

#### createElement

Vue 在 initRender 的时候挂在 \$createElement 方法，会在 render 函数里用到。

```javascript
export function initRender(vm: Component) {
  // 定义 _c， $createElement
  // 定义 $attrs 来自vm.$vnode.attrs，$options._parentVnode.attrs
  // 定义 $listeners $options._parentListeners

  const options = vm.$options;
  const parentVnode = (vm.$vnode = options._parentVnode);
  const renderContext = parentVnode && parentVnode.context;

  vm.$slots = resolveSlots(options._renderChildren, renderContext);
  vm.$scopedSlots = emptyObject;

  vm._c = (a, b, c, d) => createElement(vm, a, b, c, d, false);
  vm.$createElement = (a, b, c, d) => createElement(vm, a, b, c, d, true);

  const parentData = parentVnode && parentVnode.data;
  defineReactive(
    vm,
    "$attrs",
    (parentData && parentData.attrs) || emptyObject,
    null,
    true
  );
  defineReactive(
    vm,
    "$listeners",
    options._parentListeners || emptyObject,
    null,
    true
  );
}
```

render 里的调用这个函数，判断 tag 类型。生成 Vnode。

```javascript
function createElement( context, tag, data,children, normalizationType) {
    var vnode;
    if (tag是正常html标签) {
        vnode = new VNode(
          tag, data, children, undefined,
          undefined, context
        );
    }else if (tag 是组件) {
      // Ctor 在这里是组件的选项，后面会变成组件构造函数
      Ctor = resolveAsset(context.$options, 'components', tag)
      vnode = createComponent(Ctor, data, context, children, tag)
    }
    return vnode
}
```

组件的 Vnode 跟标签的 Vnode 是不一样的。
组件的 Vnode 是父组件与子组件传递数据的桥梁，在这里可以把它叫做组件外壳 Vnode。
组件外壳 Vnode 并不会渲染成真实 dom。
组件外壳 Vnode 里面的**\$e**l ，**Vnode.\_vnode.elm** 保存的是组件里面内容生成的真实 dom。
组件外壳 Vnode 里面的** Vnode.\_vnode** 保存的是组件里面内容生成的 Vnode。

```javascript
function createComponent(Ctor, data, context, children, tag) {
  // extractPropsFromVNodeData 作用是把传入data的 attr 中属于 props的筛选出来
  var propsData = extractPropsFromVNodeData(data, Ctor, tag);

  // nativeOn是dom事件 放到on里面，像普通标签一样处理，加在组件根标签
  const listeners = data.on;
  data.on = data.nativeOn;

  // 在组件的data选项里假如组件钩子函数
  installComponentHooks(data);

  var vnode = new VNode(
    "vue-component-" + Ctor.cid + tag,
    data,
    undefined,
    undefined,
    undefined,
    context,
    // 这个对象会做为Vnode的componentOptions，在创建节点时调用
    {
      Ctor: Ctor,
      // 父组件给子组件绑定的props
      propsData: propsData,
      // 父组件给子组件绑定的事件
      listeners: listeners,
      tag: tag,
      children: children,
    }
  );
  return vnode;
}
```

安装组件钩子函数有四个

- init
- prepatch
- insert
- destroy

```javascript
function installComponentHooks (data: VNodeData) {
  const hooks = data.hook || (data.hook = {})
  for (let i = 0; i < hooksToMerge.length; i++) {
    const key = hooksToMerge[i]
    const existing = hooks[key]
    hooks[key] = existing
  }
}

const componentVNodeHooks = {
  init (vnode) {
    // 生成组件实例，里面会new Ctor。其实就是new Vue。
      const child = vnode.componentInstance = createComponentInstanceForVnode(
        vnode,activeInstance)
    // 执行组件的mount程序，又回到组件的起点。
      child.$mount(hydrating ? vnode.elm : undefined, hydrating)
  },

  prepatch (oldVnode, vnode) {},

  insert (vnode: MountedComponentVNode) {},

  destroy (vnode: MountedComponentVNode) {}
}

Ctor 会合并 _base 其实就是Vue
export function createComponentInstanceForVnode ( vnode，){
  const options: InternalComponentOptions = {
    _isComponent: true,
    _parentVnode: vnode,
    parent
  }
  // check inline-template render functions
  const inlineTemplate = vnode.data.inlineTemplate
  if (isDef(inlineTemplate)) {
    options.render = inlineTemplate.render
    options.staticRenderFns = inlineTemplate.staticRenderFns
  }
  return new vnode.componentOptions.Ctor(options)
}
```

现在开始流程

## 生成 Vnode

假如我们有一个模板

```
<div>
	<comp></comp>
</div>
```

经过\$mount 以后，生成 render，staticRenderFns。
render 函数长什么样？

```javascript
(function anonymous() {
  with (this) {
    return _c(
      "div",
      { attrs: { id: "app" } },
      [
        _c("comp", {
          on: { test: compClick },
          nativeOn: {
            click: function ($event) {
              return aaa($event);
            },
          },
        }),
      ],
      1
    );
  }
});
```

### createElement

render 函数里的 \_c 就是 \$createElement。

### mountComponent

生成的 render 函数，在什么时候执行的呢？

```javascript
updateComponent = () => {
  vm._update(vm._render(), hydrating);
};
```

就是在 \_render 这个函数里执行的。

```javascript
Vue.prototype._render = function () {
  const vm = this;
  const { render, _parentVnode } = vm.$options;
  vm.$vnode = _parentVnode;

  try {
    // 主要代码： 调用vm上的 render 函数 生成虚拟dom
    vnode = render.call(vm._renderProxy, vm.$createElement);
  } catch (e) {}

  vnode.parent = _parentVnode;
  return vnode;
};
```

在上面的代码里，可以看到 render 被调用了，并且生成 vnode。
并且传入 vm.\$createElement。就是 render 函数里的 \_c 。

### createElm

_render 返回 Vnode。调用 \_update。在\_update 里面调用 **patch**。
_ \_patch\_\_  里面会递归调用 createElm 传入当前节点以及当前节点的 children。
根据它们的 Vnode 生成 真实 dom 节点。当然这里会区分是标签还是组件。
![image.png](https://cdn.nlark.com/yuque/0/2020/png/462392/1598428245489-1af3c9b5-f61f-4179-be42-aba81943bb30.png#align=left&display=inline&height=362&margin=%5Bobject%20Object%5D&name=image.png&originHeight=486&originWidth=796&size=34908&status=done&style=shadow&width=593)

```javascript
function createElm(
  vnode,
  insertedVnodeQueue,
  refElm,
  nested,
  ownerArray,
  index
) {
  //  判断本节点是组件还是标签
  // 是组件 会在 createComponent 里 返回true
  // 并且在这里创建组件
  if (createComponent(vnode, insertedVnodeQueue, parentElm, refElm)) {
    return;
  }

  // 下面都是 创建节点 流程
  const data = vnode.data;
  const children = vnode.children;
  const tag = vnode.tag;

  // 开始创建 真实dom。 nodeOps是dom操作包
  vnode.elm = vnode.ns
    ? nodeOps.createElementNS(vnode.ns, tag)
    : nodeOps.createElement(tag, vnode);

  // 创建孩子节点
  createChildren(vnode, children, insertedVnodeQueue);

  // 处理节点属性 class props solt等
  invokeCreateHooks(vnode, insertedVnodeQueue);

  // 把这个节点插入到父节点
  insert(parentElm, vnode.elm, refElm);
}
```

### createComponent

```javascript
function createComponent(vnode, insertedVnodeQueue, parentElm, refElm) {
  // 调用vnode.data.hook.init
  let i = vnode.data;
  if (isDef((i = i.hook)) && isDef((i = i.init))) {
    i(vnode, false /* hydrating */);
  }

  // 如果有组件实例 就initComponent 生成elm 并处理节点属性
  if (isDef(vnode.componentInstance)) {
    initComponent(vnode, insertedVnodeQueue);
    // 插入本节点
    insert(parentElm, vnode.elm, refElm);
    return true;
  }
}
```

### initComponent

```javascript
function initComponent(vnode, insertedVnodeQueue) {
  vnode.elm = vnode.componentInstance.$el;
  invokeCreateHooks(vnode, insertedVnodeQueue);
}
```

### invokeCreateHooks

invokeCreateHooks 是处理节点的属性。

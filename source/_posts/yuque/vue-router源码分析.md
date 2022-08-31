---
title: vue-router源码分析
urlname: vgtwgv
date: '2021-07-22 21:55:21 +0800'
tags: []
categories: []
---

## 相关的知识

### Window.history

Window.history 保存用户在一个**会话期间**的网站访问记录，用户每次访问一个新的 URL 即创建一个新的历史记录。
**可以看作**history 内部有一个栈，这个栈用来存放当前浏览器 session 级别（关闭前）的同一个 tab 下所有访问过的 url。但是 history 对象并没有暴露存放历史 url 的属性。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/462392/1626253670179-23d797be-2211-487e-86e4-ab6b485bac84.png#height=378&id=u05504524&margin=%5Bobject%20Object%5D&name=image.png&originHeight=254&originWidth=464&originalType=binary∶=1&size=15070&status=done&style=shadow&width=690)

- state：一个与指定网址相关的状态对象，popstate 事件触发时，该对象会传入回调函数。只读属性，不能直接修改，可以通过 pushSate 和 replaceState 操作。

- pushState： 在 history 栈中添加一个新的记录。参数：
  state： 如果不需 要这个对象，此处可以填 null
  title：现在大多数浏览器不支持或者忽略这个参数，最好用 null 代替
  url：任意有效的 URL，用于更新浏览器的地址栏，并不在乎 URL 是否已经存在地址列表中。更重要的是，它不会重新加载页面。

### window.onpopstate

popstate 事件调用 history.back()、history.forward()、history.go()等方法，会触发 popstate 事件，单纯调用 pushState()或 replaceState()不触发 popstate 事件。
访问事件的 state 属性可获取当初 pushState()或 replaceState()设置的状态数据，事件的 state 属性包含历史记录条目的 state 对象的副本。

### window.onhashchange

通过 window.location.hash 属性获取和设置 hash 值，hash 改变会触发 onhashchange 事件。
hash 值浏览器是不会随请求发送到服务器端的（即地址栏中#及以后的内容）。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/462392/1626255551788-3f28a530-c85b-48ba-bdd7-b9f2d3f50133.png#height=479&id=u81013c79&margin=%5Bobject%20Object%5D&name=image.png&originHeight=353&originWidth=391&originalType=binary∶=1&size=23049&status=done&style=shadow&width=531)
在 URL 中 传值有两方式，一种是通过 search 来传值（即 ? 后面的部分），一种是通过 hash 来传值（即 # 后面的部分）。它们之间有一个区别， 即 search 改变时浏览器会发一次的 http request，而 hash 改变时浏览器不会发送 http request。

参考：[https://blog.csdn.net/woxueliuyun/article/details/51075272](https://blog.csdn.net/woxueliuyun/article/details/51075272)

## 如何使用

直接参考官网 [https://router.vuejs.org/zh/guide/#html](https://router.vuejs.org/zh/guide/#html)

```javascript
<div id="app">
  <h1>Hello App!</h1>
  <p>
    <!-- 使用 router-link 组件来导航. -->
    <!-- 通过传入 `to` 属性指定链接. -->
    <!-- <router-link> 默认会被渲染成一个 `<a>` 标签 -->
    <router-link to="/foo">Go to Foo</router-link>
    <router-link to="/bar">Go to Bar</router-link>
  </p>
  <!-- 路由出口 -->
  <!-- 路由匹配到的组件将渲染在这里 -->
  <router-view></router-view>
</div>

// router.js
// 0. 如果使用模块化机制编程，导入Vue和VueRouter，要调用 Vue.use(VueRouter)
// 1. 定义 (router.js路由) 组件。
// 可以从其他文件 import 进来
const Foo = { template: '<div>foo</div>' }
const Bar = { template: '<div>bar</div>' }

// 2. 定义路由
// 每个路由应该映射一个组件。 其中"component" 可以是
// 通过 Vue.extend() 创建的组件构造器，
const routes = [
  { path: '/foo', component: Foo },
  { path: '/bar', component: Bar }
]

// 3. 创建 router 实例，然后传 `routes` 配置
// 你还可以传别的配置参数
const router = new VueRouter({
  routes
})

// 4. 创建和挂载根实例。通过 router 配置参数注入路由，
const app = new Vue({
  router
}).$mount('#app')

```

## 源码分析

### 第一步，创建实例

这里重点关注下 createMatcher 和 history 的创建。

- createMatcher 里面调用 createRouteMap 创建 pathList, pathMap, nameMap 保存在闭包里。并返回 match, addRoute, getRoutes, addRoutes
- history 的创建是根据 mode 来区别创建，下面会详细分析。

```javascript
export default class VueRouter {
  constructor(options: RouterOptions = {}) {
    this.app = null;
    this.apps = [];
    this.options = options;
    this.beforeHooks = [];
    this.resolveHooks = [];
    this.afterHooks = [];
    this.matcher = createMatcher(options.routes || [], this); // 创建匹配器
    // createMatcher里面调用 createRouteMap 创建pathList, pathMap, nameMap 保存在闭包里
    // 并返回 match,addRoute,getRoutes,addRoutes

    switch (
      mode // 根据mode，创建history实例
    ) {
      case "history":
        this.history = new HTML5History(this, options.base);
        break;
      case "hash":
        this.history = new HashHistory(this, options.base, this.fallback);
        break;
      case "abstract":
        this.history = new AbstractHistory(this, options.base);
        break;
      default:
        if (process.env.NODE_ENV !== "production") {
          assert(false, `invalid mode: ${mode}`);
        }
    }
  }

  init(app) {}
  onReady(cb) {}
  onError(errorCb) {}
  push() {}
  replace() {}
  go(number) {}
  back() {}
  forward() {}
  getMatchedComponents() {}
  getRoutes() {}
  addRoute(parentOrRoute) {}
  addRoutes(routes) {}
}
```

#### 创建匹配器

在 createRouteMap 内部会遍历 routes，然后每一项会经过 addRouteRecord 函数处理。addRouteRecord 会根据当前的 route 生成记录（record），这里会递归处理 children。
addRouteRecord 内部逻辑：
1 将 path 加入到 pathList 数组中
2 将 path：record 这种 key：value 加入到 pathMap
3 处理路由别名 alias。就是把别名转换成 path 再 addRouteRecord 处理
4 将 name：record。这种 key：value 加入到 nameMap

```javascript
export function createMatcher(routes, router) {
  const { pathList, pathMap, nameMap } = createRouteMap(routes);

  // addRoutes 还是调用createRouteMap 就是生成记录record，并添加到pathMap，nameMap，pathList
  function addRoutes(routes) {
    createRouteMap(routes, pathList, pathMap, nameMap);
  }
  // addRoute方法与addRoutes原理一样
  function addRoute(parentOrRoute, route) {}

  function getRoutes() {
    return pathList.map((path) => pathMap[path]);
  }

  // 根据location匹配出对应的路由记录 record，最后暴露出处理过的route
  function match(raw, currentRoute, redirectedFrom) {
    const location = normalizeLocation(raw, currentRoute, false, router);
    const { name } = location;

    if (name) {
      const record = nameMap[name];
    } else if (location.path) {
      const record = pathMap[location.path];
    }
    // 根据record返回route
    return _createRoute(record, location, redirectedFrom);
  }

  return {
    match,
    addRoute,
    getRoutes,
    addRoutes,
  };
}
```

#### 创建 history

从 new VueRouter 的代码种可以看到，会根据 mode 生成 history 实例，这里主要分析 HTML5History， HashHistory 两个类，这两个类都继承了 History 基础类。
History 类定义了 history 的一些基础功能，比如 transitionTo 跳转从此函数开始。listen 用于开启监听 route 改变触发视图更新

```javascript
// History类定义了history的一些基础功能，比如transitionTo跳转从此函数开始。listen用于开启监听route改变触发视图更新
export class History {
  constructor(router: Router, base: ?string) {
    this.router = router;
    this.base = normalizeBase(base);
    this.current = START;
    this.pending = null;
    this.ready = false;
    this.readyCbs = [];
    this.readyErrorCbs = [];
    this.errorCbs = [];
    this.listeners = [];
  }

  // 初始化完成或者跳转完成出发回调函数
  listen(cb: Function) {
    this.cb = cb;
  }

  onReady(cb: Function, errorCb: ?Function) {
    if (this.ready) {
      cb();
    } else {
      this.readyCbs.push(cb);
      if (errorCb) {
        this.readyErrorCbs.push(errorCb);
      }
    }
  }

  // 重要函数，路由跳转操作会触发此函数，内部会匹配路由，并调用跳转前确定函数
  transitionTo(location, onComplete, onAbort) {
    let route;
    route = this.router.match(location, this.current);

    const prev = this.current;

    const _onComplete = () => {
      this.updateRoute(route); // 更新route，触发更新view
      onComplete && onComplete(route);
      this.ensureURL();
      this.router.afterHooks.forEach((hook) => {
        hook && hook(route, prev);
      });

      // fire ready cbs once
      if (!this.ready) {
        this.ready = true;
        this.readyCbs.forEach((cb) => {
          cb(route);
        });
      }
    };
    const _err = (err) => {};
    this.confirmTransition(route, _onComplete, _err);
  }

  // 跳转前确定函数
  confirmTransition(route, onComplete, onAbort) {
    const current = this.current;
    this.pending = route;
    const abort = (err) => {
      if (!isNavigationFailure(err) && isError(err)) {
        if (this.errorCbs.length) {
          this.errorCbs.forEach((cb) => {
            cb(err);
          });
        } else {
          warn(false, "uncaught error during route navigation:");
          console.error(err);
        }
      }
      onAbort && onAbort(err);
    };
    const lastRouteIndex = route.matched.length - 1;
    const lastCurrentIndex = current.matched.length - 1;

    //判断如果前后是同一个路由，不进行操作
    if (
      isSameRoute(route, current) &&
      // in the case the route map has been dynamically appended to
      lastRouteIndex === lastCurrentIndex &&
      route.matched[lastRouteIndex] === current.matched[lastCurrentIndex]
    ) {
      this.ensureURL();
      return abort(createNavigationDuplicatedError(current, route));
    }

    const { updated, deactivated, activated } = resolveQueue(
      this.current.matched,
      route.matched
    );

    // 各生命周期组成的队列
    const queue: Array<?NavigationGuard> = [].concat(
      // in-component leave guards
      extractLeaveGuards(deactivated),
      // global before hooks
      this.router.beforeHooks,
      // in-component update hooks
      extractUpdateHooks(updated),
      // in-config enter guards
      activated.map((m) => m.beforeEnter),
      // async components
      resolveAsyncComponents(activated)
    );

    // 执行器，执行每一个钩子
    const iterator = (hook: NavigationGuard, next) => {
      if (this.pending !== route) {
        return abort(createNavigationCancelledError(current, route));
      }
      try {
        hook(route, current, (to: any) => {
          if (to === false) {
            // next(false) -> abort navigation, ensure current URL
            this.ensureURL(true);
            abort(createNavigationAbortedError(current, route));
          } else if (isError(to)) {
            this.ensureURL(true);
            abort(to);
          } else if (
            typeof to === "string" ||
            (typeof to === "object" &&
              (typeof to.path === "string" || typeof to.name === "string"))
          ) {
            // next('/') or next({ path: '/' }) -> redirect
            abort(createNavigationRedirectedError(current, route));
            if (typeof to === "object" && to.replace) {
              this.replace(to);
            } else {
              this.push(to);
            }
          } else {
            // confirm transition and pass on the value
            next(to);
          }
        });
      } catch (e) {
        abort(e);
      }
    };
    // 洋葱圈方式，执行钩子函数组成的数组
    runQueue(queue, iterator, () => {
      // wait until async components are resolved before
      // extracting in-component enter guards
      const enterGuards = extractEnterGuards(activated);
      const queue = enterGuards.concat(this.router.resolveHooks);
      runQueue(queue, iterator, () => {
        if (this.pending !== route) {
          return abort(createNavigationCancelledError(current, route));
        }
        this.pending = null;
        onComplete(route);
        if (this.router.app) {
          this.router.app.$nextTick(() => {
            handleRouteEntered(route);
          });
        }
      });
    });
  }

  // 更新route的函数，会触发router-view的更新，后面会细说
  updateRoute(route: Route) {
    this.current = route;
    this.cb && this.cb(route);
  }

  // 取消路由监听
  teardown() {
    this.listeners.forEach((cleanupListener) => {
      cleanupListener();
    });
    this.listeners = [];
  }
}
```

这里需要注意的是使用洋葱圈模式去异步执行路由钩子

```javascript
// 形成一个类似洋葱圈模型，一层套一层。
// queue是一个包含很多钩子的数组
// fn 是执行器，用来执行钩子
// 执行完所有钩子的回调函数
export function runQueue(queue, fn, cb) {
  const step = (index) => {
    if (index >= queue.length) {
      cb();
    } else {
      if (queue[index]) {
        fn(queue[index], () => {
          step(index + 1);
        });
      } else {
        step(index + 1);
      }
    }
  };
  step(0);
}
```

#### HTML5History

这里需要注意的是 setupListeners 启动监听，并且内部调用了 transitionTo 函数
而且 push，repalce 方法也是调用的 transitionTo 函数。
这里你可能有疑惑 History 的 listen 也是开启监听，为什么还要有 setupListeners 启动监听，这是为了应对用户直接改变 url 跳转，a 标签跳转，或者调用 push 方法跳转。
listen 是处理 route 改变的情况。
setupListeners 是监听 popstate，hashchange。

```javascript
export class HTML5History extends History {
  constructor(router: Router, base: ?string) {
    super(router, base);
    this._startLocation = getLocation(this.base);
  }

  // 启动监听器，监听 popstate 事件
  setupListeners() {
    const router = this.router;
    const expectScroll = router.options.scrollBehavior;
    const supportsScroll = supportsPushState && expectScroll;

    const handleRoutingEvent = () => {
      const current = this.current;
      const location = getLocation(this.base);

      this.transitionTo(location, (route) => {
        if (supportsScroll) {
          handleScroll(router, route, current, true);
        }
      });
    };

    window.addEventListener("popstate", handleRoutingEvent);
    this.listeners.push(() => {
      window.removeEventListener("popstate", handleRoutingEvent);
    });
  }

  go(n: number) {
    window.history.go(n);
  }

  //暴露出去的push方法 push种调用了transitionTo
  push(location: RawLocation, onComplete?: Function, onAbort?: Function) {
    const { current: fromRoute } = this;
    this.transitionTo(
      location,
      (route) => {
        //保存当前的滚动位置信息，用于返回时候复位。
        // 保留现有的历史状态。其内部还是 history.pushState/replaceState
        pushState(cleanPath(this.base + route.fullPath));
        handleScroll(this.router, route, fromRoute, false);
        onComplete && onComplete(route);
      },
      onAbort
    );
  }

  replace() {}

  ensureURL(push?: boolean) {}
}
```

#### HashHistory

HashHistory 与上面 HTML5History 逻辑基本一样，直接贴源码，这里不做分析。

```javascript
export class HashHistory extends History {
  constructor(router: Router, base: ?string, fallback: boolean) {
    super(router, base);
    // check history fallback deeplinking
    if (fallback && checkFallback(this.base)) {
      return;
    }
    ensureSlash();
  }

  // this is delayed until the app mounts
  // to avoid the hashchange listener being fired too early
  setupListeners() {
    if (this.listeners.length > 0) {
      return;
    }

    const router = this.router;
    const expectScroll = router.options.scrollBehavior;
    const supportsScroll = supportsPushState && expectScroll;

    if (supportsScroll) {
      this.listeners.push(setupScroll());
    }

    const handleRoutingEvent = () => {
      const current = this.current;
      if (!ensureSlash()) {
        return;
      }
      this.transitionTo(getHash(), (route) => {
        if (supportsScroll) {
          handleScroll(this.router, route, current, true);
        }
        if (!supportsPushState) {
          replaceHash(route.fullPath);
        }
      });
    };
    const eventType = supportsPushState ? "popstate" : "hashchange";
    window.addEventListener(eventType, handleRoutingEvent);
    this.listeners.push(() => {
      window.removeEventListener(eventType, handleRoutingEvent);
    });
  }

  push(location: RawLocation, onComplete?: Function, onAbort?: Function) {
    const { current: fromRoute } = this;
    this.transitionTo(
      location,
      (route) => {
        pushHash(route.fullPath);
        handleScroll(this.router, route, fromRoute, false);
        onComplete && onComplete(route);
      },
      onAbort
    );
  }

  replace(location: RawLocation, onComplete?: Function, onAbort?: Function) {
    const { current: fromRoute } = this;
    this.transitionTo(
      location,
      (route) => {
        replaceHash(route.fullPath);
        handleScroll(this.router, route, fromRoute, false);
        onComplete && onComplete(route);
      },
      onAbort
    );
  }

  go(n: number) {
    window.history.go(n);
  }

  ensureURL(push?: boolean) {
    const current = this.current.fullPath;
    if (getHash() !== current) {
      push ? pushHash(current) : replaceHash(current);
    }
  }

  getCurrentLocation() {
    return getHash();
  }
}
```

### 第二步，安装路由

这一过程发生在 use 阶段，目的是在 Vue 的 beforeCreate 钩子里面调用 init 开启路由初始化，并且注册 router-view，router-link 两个组件。
这里需要注意的是 Vue.util.defineReactive 的调用，在根 vue 对象设置了劫持字段\_route。修改了\_route 的 getter/settter。并收集 Watcher。详细内容参考 vue 源码解析。
当修改\_route 的时候会触发 router-view 的更新。

```javascript
export function install(Vue) {
  // 混入两个钩子，调用init
  Vue.mixin({
    beforeCreate() {
      // 调用init 初始化开始
      this._router.init(this);

      // 为根vue对象设置了劫持字段_route，用户触发router-view的变化
      Vue.util.defineReactive(this, "_route", this._router.history.current);
      registerInstance(this, this);
    },
    destroyed() {
      registerInstance(this);
    },
  });

  // 给vue原型上添加方法
  Object.defineProperty(Vue.prototype, "$router", {
    get() {
      return this._routerRoot._router;
    },
  });

  Object.defineProperty(Vue.prototype, "$route", {
    get() {
      return this._routerRoot._route;
    },
  });

  // 注册全局组件
  Vue.component("RouterView", View);
  Vue.component("RouterLink", Link);
}
```

开始初始化，保存 vue 实例，监听 hashChange，popState 事件

```javascript
  init (app) {
    this.apps.push(app) // 保存vue实例

    if (this.app) {
      return
    }
    this.app = app

    const history = this.history
    if (history instanceof HTML5History || history instanceof HashHistory) {
      const handleInitialScroll = routeOrError => {
    		// ...处理滚动条
      }
      const setupListeners = routeOrError => {
        history.setupListeners() // 监听事件
        handleInitialScroll(routeOrError)
      }
      // 调用match得到匹配的route对象，更新路由，更新url和hash
      // 调用confirmTransition来处理路由钩子
      history.transitionTo(
        history.getCurrentLocation(),
        setupListeners,
        setupListeners
      )
    }
		// 用于更新app实例上的route
    history.listen(route => {
      this.apps.forEach(app => {
        app._route = route
      })
    })
  }
```

### router-link&router-view

router-link 中处理组件的 class，切换激活状态和失活状态。绑定事件，默认 click，调用 push，repalce。还有渲染 slot 的情况。
router-view 会根据 keepAlive 对组件进行缓存，或者去 matched.components 中寻找匹配的组件。还有对组件实例的保存，在组件的 prepatch 钩子里保存到 matched.instances。还有会判断 matched.instances 用不用更新的情况，最后调用$createElement 创建组件。

```javascript
export default {
  name: 'RouterLink',
  props: {
    to: {
      type: toTypes,
      required: true
    },
    tag: {
      type: String,
      default: 'a'
    },
    custom: Boolean,
    exact: Boolean,
    exactPath: Boolean,
    append: Boolean,
    replace: Boolean,
    activeClass: String,
    exactActiveClass: String,
    ariaCurrentValue: {
      type: String,
      default: 'page'
    },
    event: {
      type: eventTypes,
      default: 'click'
    }
  },
  render (h: Function) {
    const router = this.$router
    const current = this.$route
    /解析 to的路径对应路由项
    const { location, route, href } = router.resolve(
      this.to,
      current,
      this.append
    )

    //设置一些默认元素class
    const classes = {}
    // ...

    const ariaCurrentValue = classes[exactActiveClass] ? this.ariaCurrentValue : null

    //事件处理函数,可以看到 push，repalce 方法的调用
    const handler = e => {
      if (guardEvent(e)) {
        if (this.replace) {
          router.replace(location, noop)
        } else {
          router.push(location, noop)
        }
      }
    }

    const on = { click: guardEvent }
    if (Array.isArray(this.event)) {
      this.event.forEach(e => {
        on[e] = handler
      })
    } else {
      on[this.event] = handler
    }

    const data: any = { class: classes }

    const scopedSlot = this.$scopedSlots.default({href, route,navigate,
        isActive: classes[activeClass],
        isExactActive: classes[exactActiveClass]
      })

    if (scopedSlot) {
      if (process.env.NODE_ENV !== 'production' && !this.custom) {
        !warnedCustomSlot && warn(false, 'In Vue Router 4, the v-slot API will by default wrap its content with an <a> element. Use the custom prop to remove this warning:\n<router-link v-slot="{ navigate, href }" custom></router-link>\n')
        warnedCustomSlot = true
      }
      if (scopedSlot.length === 1) {
        return scopedSlot[0]
      } else if (scopedSlot.length > 1 || !scopedSlot.length) {
        if (process.env.NODE_ENV !== 'production') {
          warn(
            false,
            `<router-link> with to="${
              this.to
            }" is trying to use a scoped slot but it didn't provide exactly one child. Wrapping the content with a span element.`
          )
        }
        return scopedSlot.length === 0 ? h() : h('span', {}, scopedSlot)
      }
    }

    if (this.tag === 'a') {
      data.on = on
      data.attrs = { href, 'aria-current': ariaCurrentValue }
    } else {
      // find the first <a> child and apply listener and href
      const a = findAnchor(this.$slots.default)
      if (a) {
        // in case the <a> is a static node
        a.isStatic = false
        const aData = (a.data = extend({}, a.data))
        aData.on = aData.on || {}
        // transform existing events in both objects into arrays so we can push later
        for (const event in aData.on) {
          const handler = aData.on[event]
          if (event in on) {
            aData.on[event] = Array.isArray(handler) ? handler : [handler]
          }
        }
        // append new listeners for router-link
        for (const event in on) {
          if (event in aData.on) {
            // on[event] is always a function
            aData.on[event].push(on[event])
          } else {
            aData.on[event] = handler
          }
        }

        const aAttrs = (a.data.attrs = extend({}, a.data.attrs))
        aAttrs.href = href
        aAttrs['aria-current'] = ariaCurrentValue
      } else {
        // doesn't have <a> child, apply listener to self
        data.on = on
      }
    }

    return h(this.tag, data, this.$slots.default)
  }
}
```

```javascript
export default {
  name: "RouterView",
  functional: true,
  props: {
    name: {
      type: String,
      default: "default",
    },
  },
  render(_, { props, children, parent, data }) {
    //直接使用父组件上下文的createElement()函数
    const h = parent.$createElement;
    const name = props.name;
    const route = parent.$route;
    const cache = parent._routerViewCache || (parent._routerViewCache = {});

    //解决router-view 嵌套问题
    let depth = 0;
    let inactive = false;
    while (parent && parent._routerRoot !== parent) {
      const vnodeData = parent.$vnode ? parent.$vnode.data : {};
      if (vnodeData.routerView) {
        depth++;
      }
      // 检查keep-alive
      if (vnodeData.keepAlive && parent._directInactive && parent._inactive) {
        inactive = true;
      }
      parent = parent.$parent;
    }
    //当前view-router的嵌套深度
    data.routerViewDepth = depth;

    // 组件缓存
    if (inactive) {
      const cachedData = cache[name];
      const cachedComponent = cachedData && cachedData.component;
      if (cachedComponent) {
        // #2301
        // pass props
        if (cachedData.configProps) {
          fillPropsinData(
            cachedComponent,
            data,
            cachedData.route,
            cachedData.configProps
          );
        }
        return h(cachedComponent, data, children);
      } else {
        // render previous empty view
        return h();
      }
    }

    const matched = route.matched[depth];
    const component = matched && matched.components[name];

    // render empty node if no matched route or no config component
    if (!matched || !component) {
      cache[name] = null;
      return h();
    }

    // cache component
    cache[name] = { component };

    // attach instance registration hook
    // this will be called in the instance's injected lifecycle hooks
    data.registerRouteInstance = (vm, val) => {
      // val could be undefined for unregistration
      const current = matched.instances[name];
      if ((val && current !== vm) || (!val && current === vm)) {
        matched.instances[name] = val;
      }
    };

    // also register instance in prepatch hook
    // in case the same component instance is reused across different routes
    (data.hook || (data.hook = {})).prepatch = (_, vnode) => {
      matched.instances[name] = vnode.componentInstance;
    };

    // register instance in init hook
    // in case kept-alive component be actived when routes changed
    data.hook.init = (vnode) => {
      if (
        vnode.data.keepAlive &&
        vnode.componentInstance &&
        vnode.componentInstance !== matched.instances[name]
      ) {
        matched.instances[name] = vnode.componentInstance;
      }

      // if the route transition has already been confirmed then we weren't
      // able to call the cbs during confirmation as the component was not
      // registered yet, so we call it here.
      handleRouteEntered(route);
    };

    const configProps = matched.props && matched.props[name];
    // save route and configProps in cache
    if (configProps) {
      extend(cache[name], {
        route,
        configProps,
      });
      fillPropsinData(component, data, route, configProps);
    }

    // 调用createElement 渲染 当前匹配到的组件
    return h(component, data, children);
  },
};
```

### push&repalce

当调用 push 方法的流程：
push----> this.history.push ----> this.transitionTo ----> this.updateRoute ----> this.cb(route) --->
app.\_route = route
当\_route 改变的时候会触发 router-view 的更新。

## 模拟 vue-router

第一步 Vue.use(Router)  的时候回去执行  install 方法
在 install 方法里面   用 mixin 的方法向 Vue 写入 beforeCreate 生命周期
为什么要在 beforeCreate  的时候插入？原因是在创建之前   为 Vue.prototype 挂载$router  方法
并触发 init 方法。

```javascript
myRouter.install = function (vm) {
  Vue = vm;
  Vue.mixin({
    beforeCreate() {
      if (this.$options.router) {
        Vue.prototype.$router = this.$options.router;
        this.$options.router.init();
      }
    },
  });
};
```

2  在 init 方法里面   触发 3 个函数
  1 监听 hashChange 事件   获取 hsah 值   并赋值给   this.app.current
  2  解析 routes  生成 routesMap
  3  为 Vue 挂载全局组件  router-link  router-view
在 new  的时候   内部创建 Vue 实例   利用 Vue 的数据响应式  this.app.current  当 current 改变的时候  router-view  就会重新渲染。

```javascript
let Vue;

class myRouter {
  constructor(options) {
    this.$options = options;
    this.routerMap = {};

    this.app = new Vue({
      data: { current: "/" },
    });

    this.init = () => {
      this.bindEvents();
      this.createRouterMap();
      this.initComponent();
    };
  }

  bindEvents() {
    window.addEventListener("hashchange", this.onHashChange.bind(this));
  }

  onHashChange() {
    this.app.current = window.location.hash.slice(1) || "/";
  }

  createRouterMap() {
    //   遍历用户配置路由数组
    this.$options.routes.forEach((route) => {
      this.routerMap[route.path] = route;
    });
  }

  initComponent() {
    Vue.component("mylink", {
      props: {
        to: String,
      },
      render(h) {
        return h("a", { attrs: { href: "#" + this.to } }, [
          this.$slots.default,
        ]);
      },
    });

    Vue.component("myview", {
      render: (h) => {
        const Component = this.routerMap[this.app.current].component;
        console.log(Component);
        return h(Component);
      },
    });
  }
}

myRouter.install = function (vm) {
  Vue = vm;

  Vue.mixin({
    beforeCreate() {
      if (this.$options.router) {
        Vue.prototype.$router = this.$options.router;
        this.$options.router.init();
      }
    },
  });
};

export default myRouter;
```

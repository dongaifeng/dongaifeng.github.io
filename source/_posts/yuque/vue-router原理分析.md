---
title: vue-router原理分析
urlname: vdc47u
date: 2019-10-30 17:04:06 +0800
tags: []
categories: []
---

第一步 Vue.use(Router)  的时候回去执行  install 方法

在 install 方法里面   用 mixin 的方法向 Vue 写入 beforeCreate 生命周期

为什么要在 beforeCreate  的时候插入？

在创建之前   为 Vue.prototype 挂载\$router  方法
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

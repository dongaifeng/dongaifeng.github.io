---
title: vuex源码分析
urlname: zmfdrd
date: '2021-06-01 17:00:16 +0800'
tags: []
categories: []
---

version: '3.5.1'
vuex 是 vue 的状态管理工具，利用 vue 的响应式原理和计算属性。
Vue.use(Vuex)的时候调用了内部的 install 方法，

## install

```javascript
export function install(_Vue) {
  // Vue 已经存在并且相等，说明已经Vuex.use过
  if (Vue && _Vue === Vue) {
    // 省略代码：非生产环境报错，vuex已经安装
    return;
  }
  Vue = _Vue;
  applyMixin(Vue);
}
```

我们只看主要代码，install 又调用了 applyMixin。applyMixin 里面很简单，只是在 vue 的 beforeCreate 钩子函数里混入了 vuexInit 方法。

```javascript
function vuexInit() {
  var options = this.$options;
  // store injection
  if (options.store) {
    this.$store =
      typeof options.store === "function" ? options.store() : options.store;
  } else if (options.parent && options.parent.$store) {
    this.$store = options.parent.$store;
  }
}
```

vuexInit 逻辑也很清除 在 vue 的实例上改在$store，也就是在 new Vue 的时候传入的 store。
下面我们来看 new Vuex.store 的时候做了什么事。

## Store

```javascript
var Store = function Store(options) {
  var this$1 = this;
  if (options === void 0) options = {};

  // cnd引入时，不会use(Vue),所以用这种方式获取Vue
  if (!Vue && typeof window !== "undefined" && window.Vue) {
    install(window.Vue);
  }

  this._committing = false;
  this._actions = Object.create(null); // 用来存放处理后的用户自定义的actoins
  this._actionSubscribers = [];
  this._mutations = Object.create(null); // 用来存放处理后的用户自定义的mutations
  this._wrappedGetters = Object.create(null); // 用来存放处理后的用户自定义的 getters
  this._modules = new ModuleCollection(options); // 构建模块树结构
  this._modulesNamespaceMap = Object.create(null); // 模块名对应模块的map
  this._subscribers = [];
  this._watcherVM = new Vue();
  this._makeLocalGettersCache = Object.create(null);

  // 定义dispatch， commit方法。使this指向store
  this.dispatch = function boundDispatch(type, payload) {
    return dispatch.call(store, type, payload);
  };
  this.commit = function boundCommit(type, payload, options) {
    return commit.call(store, type, payload, options);
  };

  // strict mode
  this.strict = strict;
  var state = this._modules.root.state;

  // 构建_modulesNamespaceMap，使key为模块的namespace，value为模块。
  // 构建root的state与child的state的树形结构
  // 用Vue.set在root的设置子模块属性
  // 遍历mutation， action， getter， child。依次安装
  installModule(this, state, [], this._modules.root);

  // 遍历_wrappedGetters生成computed，并且把state作为data，new Vue生成实例。
  // 使state变成响应式
  // 并用的Object.defineProperty代理store.getters到store._vm上。
  resetStoreVM(this, state);

  // apply plugins
  plugins.forEach(function (plugin) {
    return plugin(this$1);
  });

  var useDevtools =
    options.devtools !== undefined ? options.devtools : Vue.config.devtools;
  if (useDevtools) {
    devtoolPlugin(this);
  }
};
```

通过上面的代码，可以看出主要逻辑是在 ModuleCollection， installModule， resetStoreVM 三个函数中。
下面来逐个分析。

## ModuleCollection

模块收集器，听名字就知道是收集模块的，构建成一个模块树。

```javascript
var ModuleCollection = function ModuleCollection(rawRootModule) {
  this.register([], rawRootModule, false);
};

/**
 * 注册模块
 * @param {Array} path 路径
 * @param {Object} rawModule 原始未加工的模块
 * @param {Boolean} runtime runtime 默认是 true
 */
ModuleCollection.prototype.register = function register(
  path,
  rawModule,
  runtime
) {
  var this$1 = this;
  // 生成module模块。
  var newModule = new Module(rawModule, runtime);
  // 形成父子嵌套的module链
  if (path.length === 0) {
    this.root = newModule;
  } else {
    var parent = this.get(path.slice(0, -1));
    parent.addChild(path[path.length - 1], newModule);
  }

  // 递归 所有的子模块, 注意这里 path.concat(key) 的变化
  if (rawModule.modules) {
    forEachValue(rawModule.modules, function (rawChildModule, key) {
      this$1.register(path.concat(key), rawChildModule, runtime);
    });
  }
};

// 模块的生成，包含了了一些对_children的添加删除操作的方法，
// 遍历getter，action，mutation的遍历方法等
var Module = function Module(rawModule, runtime) {
  this.runtime = runtime;
  // 子模块
  this._children = Object.create(null);
  // 存储原始未加工的模块
  this._rawModule = rawModule;
  var rawState = rawModule.state;

  // 原始Store 可能是函数，也可能是是对象，是假值，则赋值空对象。
  this.state = (typeof rawState === "function" ? rawState() : rawState) || {};
};
Module.prototype.addChild = function addChild(key, module) {
  this._children[key] = module;
};
```

Module 生成一个模块的基本结构，包含模块的原始数据和处理后的数据，还有一些对 getters，actions 的遍历方法，还有对子模块的增删操作。
ModuleCollection 的作用就是递归遍历各个模块，根据 path 形成一种父子嵌套的结构。

## installModule

注册每个模块的 actions， motations，getters， 并在根组件响应式挂在各个子模块。

```javascript
function installModule(store, rootState, path, module, hot) {
  var isRoot = !path.length;
  var namespace = store._modules.getNamespace(path);

  // 注册一个map：_modulesNamespaceMap 使模块名 = 模块。这里会有重名判断
  if (module.namespaced) {
    store._modulesNamespaceMap[namespace] = module;
  }

  // 运用Vue.set响应式的在根模块上设置子模块属性
  if (!isRoot && !hot) {
    var parentState = getNestedState(rootState, path.slice(0, -1));
    var moduleName = path[path.length - 1];
    store._withCommit(function () {
      Vue.set(parentState, moduleName, module.state);
    });
  }

  var local = (module.context = makeLocalContext(store, namespace, path));

  // 遍历mutations
  module.forEachMutation(function (mutation, key) {
    var namespacedType = namespace + key;
    registerMutation(store, namespacedType, mutation, local);
  });
  // ...actions， getters
}

// 向store._mutations添加方法。
function registerMutation(store, type, handler, local) {
  var entry = store._mutations[type] || (store._mutations[type] = []);
  entry.push(function wrappedMutationHandler(payload) {
    handler.call(store, local.state, payload);
  });
}
```

module.context  这个赋值主要是给  helpers  中  mapState、mapGetters、mapMutations、mapActions 四个辅助函数使用的。生成组件的 dispatch、commit、getters 和 state。

## resetStoreVM

这个函数做的是重置 storeVM，就是重置作为 store 使用的 vue 实例。在第一次 new Store 的时候用作初始化 storeVM。

```javascript
function resetStoreVM(store, state, hot) {
  var oldVm = store._vm;

  store.getters = {};
  // 重置 本地getters的缓存
  store._makeLocalGettersCache = Object.create(null);
  var wrappedGetters = store._wrappedGetters;
  var computed = {};

  // 遍历getters，代理到vm实例上
  forEachValue(wrappedGetters, function (fn, key) {
    computed[key] = partial(fn, store);
    Object.defineProperty(store.getters, key, {
      get: function () {
        return store._vm[key];
      },
      enumerable: true, // for local getters
    });
  });

  // state作为实例的data，getters作书实例的计算属性，生成Vue实例，实现响应式
  var silent = Vue.config.silent;
  Vue.config.silent = true;
  store._vm = new Vue({
    data: {
      $$state: state,
    },
    computed: computed,
  });
  Vue.config.silent = silent;

  // enable strict mode for new vm
  if (store.strict) {
    enableStrictMode(store);
  }

  // 如果有旧的vm实例，就销毁。先生成新的再销毁。
  if (oldVm) {
    if (hot) {
      store._withCommit(function () {
        oldVm._data.$$state = null;
      });
    }
    Vue.nextTick(function () {
      return oldVm.$destroy();
    });
  }
}
```

从这个函数我们可以看出 vuex 就是利用 vue 的响应式，getter 利用的就是 computed 计算属性。
computed 依赖他的依赖项进行缓存和更新。
data 改变会通过组件的 watcher 进行更新。

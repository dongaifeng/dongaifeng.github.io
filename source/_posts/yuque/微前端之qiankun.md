---
title: 微前端之qiankun
urlname: gk8a1p
date: '2021-05-28 10:12:11 +0800'
tags: []
categories: []
---

## 使用

参考官网,不多说了。。。

## 原理

### 沙箱机制

**ProxySandbox**
当调用 set 向子应用 proxy/window 对象设置属性时，所有的属性设置和更新都会命中 updateValueMap，存储在 updateValueMap 集合中（第 38 行），从而避免对 window 对象产生影响（旧版本则是通过 diff 算法还原 window 对象状态快照，子应用之间的状态是隔离的，而父子应用之间 window 对象会有污染）。
当调用 get 从子应用 proxy/window 对象取值时，会优先从子应用的沙箱状态池 updateValueMap 中取值，如果没有命中才从主应用的 window 对象中取值（第 49 行）。对于非构造函数的取值将会对 this 指针绑定到 window 对象后，再返回函数。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/462392/1618913257330-76b8ae8c-d10c-41c0-9949-357c784ce918.png#clientId=u359d20c0-691c-4&from=paste&height=756&id=u854065ba&margin=%5Bobject%20Object%5D&name=image.png&originHeight=756&originWidth=950&originalType=binary∶=1&size=513245&status=done&style=none&taskId=u12a1c44b-b0b4-4007-93d1-011aa76c5c2&width=950)
**SnapshotSandbox**
我们先看 active 函数，在沙箱激活时，会先给当前 window 对象打一个快照，记录沙箱激活前的状态（第 38~40 行）。打完快照后，函数内部将 window 状态通过 modifyPropsMap 记录还原到上次的沙箱运行环境，也就是还原沙箱激活期间（历史记录）修改过的 window 属性。
在沙箱关闭时，调用 inactive 函数，在沙箱关闭前通过遍历比较每一个属性，将被改变的 window 对象属性值（第 54 行）记录在 modifyPropsMap 集合中。在记录了 modifyPropsMap 后，将 window 对象通过快照 windowSnapshot 还原到被沙箱激活前的状态（第 55 行），相当于是将子应用运行期间对 window 造成的污染全部清除。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/462392/1618913522875-f188f5bc-9bda-434d-8126-cbc5e0293790.png#clientId=u359d20c0-691c-4&from=paste&height=675&id=u6d08be20&margin=%5Bobject%20Object%5D&name=image.png&originHeight=675&originWidth=947&originalType=binary∶=1&size=234001&status=done&style=none&taskId=u44a017c5-3406-499c-91e3-6efa02cb27d&width=947)

### 全局状态

### import-html-entry

通过 http 请求加载指定地址的首屏内容即 html 页面，然后解析这个 html 模版得到 template,scripts,entry,styles。
htmlEntry 向外暴露出一个 promise。如下：

```javascript
{
  // template 是 link 替换为 style 后的 template
	template: embedHTML,
	// 静态资源地址
	assetPublicPath,
	// 获取外部脚本，最终得到所有脚本的代码内容
	getExternalScripts: () => getExternalScripts(scripts, fetch),
	// 获取外部样式文件的内容
	getExternalStyleSheets: () => getExternalStyleSheets(styles, fetch),
	// 脚本执行器，让 JS 代码(scripts)在指定 上下文 中运行
	execScripts: (proxy, strictGlobal) => {
		if (!scripts.length) {
			return Promise.resolve();
		}
		return execScripts(entry, scripts, proxy, { fetch, strictGlobal });
	}
}
```

qiankun 就用了这个对象中的 template、assetPublicPath 和 execScripts 三项，将 template 通过 DOM 操作添加到主应用中，执行 execScripts 方法得到微应用导出的生命周期方法，并且还顺便解决了 JS 全局污染的问题，因为执行 execScripts 方法的时候可以通过 proxy 参数指定 JS 的执行上下文。
execScripts(scripts, fetch, error) 函数执行时：获取指定的所有外部脚本的内容也就是 scripts，并设置每个脚本的执行上下文 global，然后通过 eval 函数运行

## 源码分析

registerMicroApps

1. 注册子 app，过滤 apps 数组，过滤掉已经注册的 app
1. 遍历没有注册的 app 数组，每个执行 single-spa 的 registerApplication 方法

```javascript
注册子app
export function registerMicroApps<T extends ObjectType>(apps,lifeCycles) {
  // 过滤掉已经注册的app
  const unregisteredApps = apps.filter((app) => !microApps.some((registeredApp) => registeredApp.name === app.name));
  microApps = [...microApps, ...unregisteredApps];

  // 注册 没有注册的app
  unregisteredApps.forEach((app) => {
    const { name, activeRule, loader = noop, props, ...appConfig } = app;

    // 调用 single-spa 的 registerApplication 方法注册微应用
    registerApplication({
      name,
      app: async () => {  // 微应用的加载方法，Promise<生命周期方法组成的对象>
        loader(true); // 显示loading
        // 这句可以忽略，目的是在 single-spa 执行这个加载方法时让出线程，让其它微应用的加载方法都开始执行
        await frameworkStartedDefer.promise;

        // 返回 bootstrap、mount、unmount、update 这个几个生命周期。还有name
        const { mount, ...otherMicroAppConfigs } = (
          await loadApp({ name, props, ...appConfig }, frameworkConfiguration, lifeCycles)
        )();

        return {
          mount: [async () => loader(true), ...toArray(mount), async () => loader(false)],
          ...otherMicroAppConfigs,
        };
      },
      activeWhen: activeRule,
      customProps: props,
    });
  });
}
```

start

1. 合并配置项，预设开启预加载，单例模式，开启沙箱
1. 开启预加载
1. 开启沙箱
1. 执行 single-spa 的 start 方法

```javascript
// 启动函数
export function start(opts: FrameworkConfiguration = {}) {
  // 框架默认开启预加载、单例模式、样式沙箱
  frameworkConfiguration = {
    prefetch: true,
    singular: true,
    sandbox: true,
    ...opts,
  };
  const { prefetch, sandbox, singular, urlRerouteOnly, ...importEntryOpts } =
    frameworkConfiguration;

  // 预加载
  if (prefetch) {
    // 执行预加载策略，参数分别为微应用列表、预加载策略、{ fetch、getPublicPath、getTemplate }
    doPrefetchStrategy(microApps, prefetch, importEntryOpts);
  }

  // 开启沙箱，如果不支持Proxy，使用快照沙箱。快照沙箱只能用于单例，所以没有设置单例，会提醒
  if (sandbox) {
    if (!window.Proxy) {
      console.warn(
        "[qiankun] Miss window.Proxy, proxySandbox will degenerate into snapshotSandbox"
      );
      frameworkConfiguration.sandbox =
        typeof sandbox === "object"
          ? { ...sandbox, loose: true }
          : { loose: true };
      if (!singular) {
        console.warn(
          "[qiankun] Setting singular as false may cause unexpected behavior while your browser not support window.Proxy"
        );
      }
    }
  }

  // 执行 single-spa 的 start 方法，启动 single-spa
  startSingleSpa({ urlRerouteOnly });

  frameworkStartedDefer.resolve();
}
```

## single-spa

## 总结

##### 参考文章

- [https://juejin.cn/post/6885211340999229454#heading-27](https://juejin.cn/post/6885211340999229454#heading-27)
- [https://mp.weixin.qq.com/s/ZDO_VXFeZw6oGt1rqRRjOQ](https://mp.weixin.qq.com/s/ZDO_VXFeZw6oGt1rqRRjOQ)

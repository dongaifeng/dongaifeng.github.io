---
title: keep-alive
urlname: oqxw1i
date: '2021-06-01 17:00:16 +0800'
tags: []
categories: []
---

```javascript
/* 检测name是否匹配 规则*/
function matches(
  pattern: string | RegExp | Array<string>,
  name: string
): boolean {
  if (Array.isArray(pattern)) {
    return pattern.indexOf(name) > -1;
  } else if (typeof pattern === "string") {
    return pattern.split(",").indexOf(name) > -1;
  } else if (isRegExp(pattern)) {
    return pattern.test(name);
  }
  /* istanbul ignore next */
  return false;
}

/* 根据 规则 修正cache */
function pruneCache(keepAliveInstance: any, filter: Function) {
  const { cache, keys, _vnode } = keepAliveInstance;
  for (const key in cache) {
    const cachedNode: ?VNode = cache[key];
    if (cachedNode) {
      const name: ?string = getComponentName(cachedNode.componentOptions);
      if (name && !filter(name)) {
        pruneCacheEntry(cache, key, keys, _vnode);
      }
    }
  }
}

/* 销毁vnode对应的组件实例（Vue实例） */
function pruneCacheEntry(
  cache: VNodeCache,
  key: string,
  keys: Array<string>,
  current?: VNode
) {
  const cached = cache[key];
  if (cached && (!current || cached.tag !== current.tag)) {
    cached.componentInstance.$destroy();
  }
  cache[key] = null;
  remove(keys, key);
}

export default {
  name: "keep-alive",
  abstract: true,

  props: {
    include: patternTypes,
    exclude: patternTypes,
    max: [String, Number],
  },

  created() {
    this.cache = Object.create(null);
    this.keys = [];
  },

  // 遍历this.cache对象，然后将那些被缓存的并且当前没有处于被渲染状态的组件都销毁掉
  destroyed() {
    for (const key in this.cache) {
      pruneCacheEntry(this.cache, key, this.keys);
    }
  },

  // 监听include， exclude 当其发生变化时，重新 匹配缓存规则
  mounted() {
    this.$watch("include", (val) => {
      pruneCache(this, (name) => matches(val, name));
    });
    this.$watch("exclude", (val) => {
      pruneCache(this, (name) => !matches(val, name));
    });
  },

  render() {
    const slot = this.$slots.default; /* 获取默认插槽中的第一个组件节点 */
    const vnode: VNode = getFirstComponentChild(slot);
    const componentOptions: ?VNodeComponentOptions =
      vnode && vnode.componentOptions;
    if (componentOptions) {
      // check pattern
      const name: ?string =
        getComponentName(componentOptions); /* 获取该组件节点的名称 */
      const { include, exclude } = this;
      /* 如果name与include规则不匹配或者与exclude规则匹配则表示不缓存，直接返回vnode */
      if (
        // not included
        (include && (!name || !matches(include, name))) ||
        // excluded
        (exclude && name && matches(exclude, name))
      ) {
        return vnode;
      }

      const { cache, keys } = this;
      const key: ?string =
        vnode.key == null /* 获取组件的key */
          ? // same constructor may get registered as different local components
            // so cid alone is not enough (#3269)
            componentOptions.Ctor.cid +
            (componentOptions.tag ? `::${componentOptions.tag}` : "")
          : vnode.key;
      if (cache[key]) {
        /* 如果命中缓存，则直接从缓存中拿 vnode 的组件实例 */
        vnode.componentInstance = cache[key].componentInstance;
        // make current key freshest
        /* 调整该组件key的顺序，将其从原来的地方删掉并重新放在最后一个 */
        remove(keys, key);
        keys.push(key);
      } else {
        /* 如果没有命中缓存，则将其设置进缓存 */
        cache[key] = vnode;
        keys.push(key);
        // prune oldest entry
        /* 如果配置了max并且缓存的长度超过了this.max，则从缓存中删除第一个 */
        if (this.max && keys.length > parseInt(this.max)) {
          pruneCacheEntry(cache, keys[0], keys, this._vnode);
        }
      }

      /* 最后设置keepAlive标记位，用于 activated， deactivated的触发判断条件*/
      vnode.data.keepAlive = true;
    }
    return vnode || (slot && slot[0]);
  },
};
```

keep=alive 里面使用了**LRU 淘汰缓存策略**
如果数据最近被访问过，所以我们将新数据插入到缓存队列的尾部，当缓存数量达到最大值时，就删除缓存队列里面的第一个数据。这也就实现不常访问的数据会被从缓存队列里去除。

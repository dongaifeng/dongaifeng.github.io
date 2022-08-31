---
title: vue.config.js
urlname: ziyzct
date: '2020-09-08 18:58:37 +0800'
tags: []
categories: []
---

```javascript
module.exports = {
  publicPath: "/",
  devServer: {}, // 开发设置
  configureWebpack: {
    // weboack设置
    name: "很好", // 这里定义的name 可以在html中使用 <title><%= webpackConfig.name %></title>
  },
  chainWebpack(config) {
    // 对webpack  的config 链式擦做 修改 loader plugins
  },
};
```

vue inspect --rules  查看 webpack  加载规则  loader
vue inspect --rule svg  查看 svg  的规则

### 自定义图标

1  安装  svg 的 loader 
npm i svg-sprite-liader -D
2  修改 svg 规则

```javascript
  chainWebpack(config){
     // 对config进行链式操作即可修改loader、plugins
        // 1.svg rule中要排除icons目录
        console.log(resolve('src/icons'));

        config.module.rule('svg')
            .exclude.add(resolve('src/icons'))

        // 2.添加一个规则icons
        config.module.rule('icons')
            .test(/\.svg$/)
            .include.add(resolve('src/icons')).end()
            .use('svg-sprite-loader')
            .loader('svg-sprite-loader')
            .options({symbolId: 'icon-[name]'})
  }
```

3  创建 icons 文件夹   创建文件 index,js

```javascript
import Vue from "vue";
import icon from "./svgIcon";

//  指定 ./svg 目录下 所有.svg文件 返回数组  里面是文件名
const req = require.context("./svg", false, /\.svg$/);

req.keys().map(req);
Vue.component("icon", icon);
```

4  穿件 SvgIcon  组件

```javascript
<template>
  <svg :class="svgClass" aria-hidden="true" v-on="$listeners">
    <use :xlink:href="iconName" />
  </svg>
</template>
<script>
export default {
  name: 'SvgIcon',
  props: {
    iconClass: { type: String, required: true },
    className: { type: String, default: '' }
  },
  computed: {
    iconName() {
      return `#icon-${this.iconClass}`
    },
    svgClass() {
      if (this.className) {
        return 'svg-icon ' + this.className
      } else {
        return 'svg-icon'
      }
    }
  }
}
</script> <style scoped>
.svg-icon {
  width: 1em;
  height: 1em;
  vertical-align: -0.15em;
  fill: currentColor;
  overflow: hidden;
}
</style>
```

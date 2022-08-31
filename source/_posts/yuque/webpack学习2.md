---
title: webpack学习2
urlname: dxy6gr
date: '2020-09-08 19:00:22 +0800'
tags: []
categories: []
---

## 优化

#### 优化 loader 配置

include exclude 字段来缩小 loader 范围

#### 优化 resolve.modules 配置

resolve.modules ⽤于配置 webpack 去哪些⽬录下寻找第三⽅模块，默认是
['node_modules']寻找第三⽅模块，默认是在当前项⽬⽬录下的 node_modules ⾥⾯去找，如果没有找
到，就会去上⼀级⽬录../node_modules 找。

```javascript
module.exports = {
  resolve: {
    modules: [path.resolve(__dirname, "./node_modules")],
  },
};
```

#### 优化 resolve.alias 配置

#### 优化 resolve.extensions 配置

#### 使⽤静态资源路径 publicPath(CDN)

### css ⽂件的处理

#### 借助 MiniCssExtractPlugin 完成抽离 css

#### 压缩 css

#### 压缩 HTML

#### 优化⽂件监听的性能

## 模式区分打包

#### webpack-merge

#### 基于环境变量区分

##  tree Shaking

### 副作⽤

### 代码分割 code Splitting

### DllPlugin 插件打包第三⽅类库 优化构建性能

### 使⽤ happypack 并发执⾏任务

### 多⼊⼝打包配置通⽤⽅案

## 如何⾃⼰编写⼀个 Loader

## 如何⾃⼰编写⼀个 Plugin

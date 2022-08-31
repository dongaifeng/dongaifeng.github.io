---
title: npm 插件发布
urlname: vu6qhs
date: '2019-12-17 09:37:40 +0800'
tags: []
categories: []
---

### 使用脚本发布

```javascript
#!/usr/bin/env bash
npm config get registry # 检查仓库镜像库
npm config set registry=http://registry.npmjs.org
echo '请进⾏登录相关操作：'
npm login # 登陆
echo "-------publishing-------"
npm publish # 发布
npm config set registry=https://registry.npm.taobao.org # 设置为淘宝镜像
echo "发布完成"
exit
```

### 版本更新

```javascript
// major：主版本号
// minor：次版本号
// patch：补丁号
npm version patch
npm publish
```

### 常用命令

```javascript
// 初始化后会出现一个package.json配置文件
npm init

//安装的包只用于开发环境，不用于生产环境
npm install 包名 --save-dev
npm install 包名 -D

// 安装的包需要发布到生产环境的
npm install 包名 --save
npm install 包名 -S

// 安装选择源工具包
npm install nrm -g
nrm ls
nrm test
nrm use taobao
```

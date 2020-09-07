---
title: 开篇笔记~~~
tags: vue
category: 日记
date: 2019-09-24 16:48:56
---

出差去杭州，夜车卧铺，十几个小时.有点无聊，就搞一个小博客吧。
原来[CSDN](https://blog.csdn.net/weixin_39398244)上也写了一些博文。有兴趣的可以去看下，不过有些乱。
下面说一下基于 hexo 建立个人博客的步骤吧！

## 安装 hexo

### 先创建一个文件夹 blog，然后 cd 到这个文件夹下，输入下面命令。

```bash
$ npm install -g hexo-cli"
```

### 初始化一下 hexo

```bash
$ hexo init myblog
```

### 安装依赖

```bash
$ cd myblog
$ npm install
```

### 运行项目

```bash
$ hexo g
$ hexo server
```

## gitHub 建立仓库

### 创建

#### 创建一个和你用户名相同的仓库，后面加.github.io，只有这样，将来要部署到 GitHub page 的时候，才会被识别，也就是 xxxx.github.io，其中 xxx 就是你注册 GitHub 的用户名。

### 修改配置文件 config.yml，

```
deploy:
  type: git
  repo: https://github.com/YourgithubName/YourgithubName.github.io.git
  branch: master
```

## 部署项目

```bash
$ hexo clean
$ hexo generate
$ hexo deploy
```

详细步骤看这里: [Deployment](https://hexo.io/docs/deployment.html)

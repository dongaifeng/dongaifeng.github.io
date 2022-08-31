---
title: webpack学习1
urlname: idx6bb
date: '2020-09-08 19:00:22 +0800'
tags: []
categories: []
---

### 安装

npm install webpack webpack-cli -D
不推荐全局安装，这会使你的项目 webpack 锁定版本，造成 webpack 依赖因与 webpack 版本不同产生冲突。

### 特性

- webpack 默认只支持 js，json。
- 支持 comminJs，es6 的 moudule，AMD 等模块。

### 启动

项目安装启动方式：

1.  ./node_modules/.bin/webpack
1.  script 中添加命令： "dev": "webpack"
1.  npx webpack 方式启动

### 配置项 webpack.config.js

#### entry  入口

入口文件有单入口，多入口方式，默认是：‘./src/index.js'

```javascript
// 单入口
entry:"./src/index.js"
// 多入口
entry:{
 index:"./src/index.js",
 login:"./src/login.js"
}
```

#### output  出口

打包压缩后的文件输出的位置，默认 './dist/main.js'

```javascript
output: {
 filename: "bundle.js",// 输出文件的名称
 path: path.resolve(__dirname, "dist")// 输出文件到磁盘的目录，必须是绝对路径
},

// 多入口
output: {
 filename: "[name][chunkhash:8].js",//利用占位符
 path: path.resolve(__dirname, "dist")
},
```

webpack 的占位符：

- [name]
- [chunkhash:8]
- [hash]
- [contexthash:6]

#### mode  环境

production, development, none

#### module  模块

module.rules  配置 loader 规则
[各种 loader 的使用方法](https://webpack.docschina.org/loaders/)
loader 的使用：

```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.(png|jpe?g|gif)$/, // 用正则匹配要使用这个loader的文件
        use: {
          loader: "file-loader",
          options: {
            // 这个loader的选项，每个loader有各自的选项
            name: "[name]_[hash].[ext]", // 输出文件的名称
            outputPath: "img/", // 存放目录 把output当做根目录
          },
        },
      },

      // 对css文件使用的loader
      {
        test: /\.css$/,
        exclude: "", //
        include: "",
        use: ["style-loader", "css-loader"], // 使用多个loader 执行顺序从右到左 从上到下
      },
    ],
  },
};
```

#### plugins  插件配置

piugin 可以在 webpack 运行到某个阶段，做一些事情，类似生命周期
[webpack 的各种 plugin 使用方法](https://webpack.docschina.org/plugins/html-webpack-plugin/)

```javascript
 plugins: [ // webpack的插架都是需要new的 参数是插件的配置项
    new htmlWebpackPlugin({ // 使用htmlWebpackPlugin插件
      title: 'my demo',
      filename: 'app.html',
      template: './src/index.html'
    }),
    new CleanWebpackPlugin()
  ]

// index.html
<title><%= htmlWebpackPlugin.options.title %></title>
```

#### sourceMap

源代码与打包后的映射关系，通过 sourceMap 定位到源码。

```javascript
devtool: "cheap-module-eval-source-map"; // 开发环境
```

#### WebpackDevServer

每次改完代码都需要重新打包⼀次，打开浏览器，刷新⼀次，很麻烦,我们可以安装
使⽤ webpackdevserver 来改善这块的体验
安装：npm install webpack-dev-server -D
配置：

```javascript
"scripts": {
 "server": "webpack-dev-server"
 },
```

```javascript
devServer: {
    contentBase: './dist', // 告诉服务器从那个目录获取内容

    open: true, // 是否打开浏览器

    port: 8080, // 端口

    after: function(app, server) {},  // 往服务器加一个中间件， 加到所有中间件最后

    before: function(app, server) {},  // 往服务器加一个中间件， 加到所有中间件最前

    // 如果要请求后台接口会出现跨域，可以将请求前面加个/api, 然后devServer会截获每个请求里的/api 并把他替换成 后端的ip
    proxy: {  // 将项目里的请求 代理 到 8080的服务器
      'api': {
        target: 'http://localhost:8080',
        pathRewrite: { '^/api': ''}  // 所有接口中有 /api 的 全部替换成 '',意思就是删除 /api
      }
    },

    publicPath: '/assets/', // 服务器静态资源访问路径

    hotOnly: true // 开启热模块更新
  }
```

#### HMR 热模块替换

#### Babel

Babel 是 JavaScript 编译器，能将 ES6 代码转换成 ES5 代码.
Babel 在执⾏编译的过程中，会从项⽬根⽬录下的 .babelrc JSON ⽂件中读取配置。
没有该⽂件会从 loader 的 options 地⽅读取配置。
安装：npm i babel-loader @babel/core @babel/preset-env -D
1.babel-loader 是 webpack 与 babel 的通信桥梁，不会做把 es6 转成 es5 的⼯作，
这部分⼯作需要⽤到@babel/preset-env 来做
2.@babel/preset-env ⾥包含了 es，6，7，8 转 es5 的转换规则。

```javascript
//index.js 顶部
import "@babel/polyfill";
```

```javascript
{
 test: /\.js$/,
 exclude: /node_modules/,
 use: {
 loader: "babel-loader",
 options: {
 presets: ["@babel/preset-env"]
 }
 }
}
```

通过上⾯的⼏步 还不够，默认的 Babel 只⽀持 let 等⼀些基础的特性转换，Promise 等
⼀些还有转换过来，这时候需要借助@babel/polyfill，把 es 的新特性都装进来，来弥
补低版本浏览器中缺失的特性
npm install --save @babel/polyfill

```javascript
 {
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: "babel-loader",
          options: {
            presets: [
              [
                "@babel/preset-env",
                {
                  targets: {
                    edge: "17",
                    firefox: "60",
                    chrome: "67",
                    safari: "11.1"
                  },
                  corejs: 2,//新版本需要指定核⼼库版本
                  useBuiltIns: "usage"//按需注⼊
                }
              ]
            ]
          }
        }
      }
```

或者新建.babelrc ⽂件，把 options 部分移⼊到该⽂件中

#### 课堂杂项

npm run  会创建一个 shell 脚本，这个脚本会把当前项目的 node_modules 的绝对路径放到系统的环境变量里，所以它才能执行。
env 命令：查看环境变量。

```javascript
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "predev": "rimraf ./dist", // predev dev前置命令 必须跟dev一样
    "dev": "webpack",
    "postdev": "cd ./dist && touch index.html" // postdev dev的后置目录
  },
```

&&  先执行完上一个   再执行下一个
&  前后两个命令同时执行
cross-env 模块   是在 win 下兼容 linux 命令。
npm script  传参：

```javascript
"dev": "Node_env=prod webpack",
  process.env.Node_env // js文件中获取
```

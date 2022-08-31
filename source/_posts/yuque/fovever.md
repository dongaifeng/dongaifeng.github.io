---
title: fovever
urlname: ww7aph
date: '2019-10-28 16:37:02 +0800'
tags: []
categories: []
---

一个用来持续（或者说永远）运行一个给定脚本的简单的命令行工具

[fovever 的 github](https://github.com/nodejitsu/forever)

安装
npm i forever -g

简单的启动
forever start app.js

关闭应用
forever stop app.js

重启所有应用
forever restartall

监听当前文件夹下所有文件改动
forever start -w app.js

显示所有运行的服务
forever list

停止所有服务
forever stopall

停止服务（使用 id）
forever stop id

判断环境变量
NODE_ENV=development forever start -l forever.log -e err.log -a app.js

指定 forever 信息输出文件，当然，默认它会放到~/.forever/forever.log
forever start -l forever.log app.js

指定 app.js 中的日志信息和错误日志输出文件
-o  就是 console.log 输出的信息，-e  就是 console.error 输出的信息
forever start -o out.log -e err.log app.js

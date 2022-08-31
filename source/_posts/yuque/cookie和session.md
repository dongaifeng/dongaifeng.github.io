---
title: cookie和session
urlname: ww2dzu
date: '2020-09-08 18:59:14 +0800'
tags: []
categories: []
---

最近在维护公司前辈留下来的 node 项目，WTF！！！  真不是一般的闹心。一个项目居然有两个登陆页，劳资搞了半天才发现从路由里 render 了另一个 index.ejs。一问才知道这项目被别人改了无数次，里面的线路早就成迷宫了。

在看到前辈大神用 cookie，session 来做用户验证这块，甚是难懂。只怪自己基础没有打牢固。一气之下从头学习 cookie 和 session。所以才有了这篇笔记。略粗糙，哈哈哈。

都知道 http 协议是无状态协议。啥意思呢？
客户端请求服务端，一旦数据交换完毕，客户端与服务端的连接就会关闭，再次交换数据需要建立新的连接。既然是新的连接，服务器当然就不会知道上一次是谁跟我连接了。这就意味着服务器无法从连接上跟踪会话。即无状态。

这很不友好，所以 cookie 就产生。据说是大网景的前员工做的。
cookie 是由服务器生成，保存在客户端的一种信息载体。这个载体存放用户信息（可以看做客户端与服务器的会话状态。
只要 cookie 没有失效   那么保存其中的会话状态就有效。失效的意思是关闭网页了或者 cookie 超时了。

用户第一次提交连接服务器，由服务器生成 cookie  并将其封装在响应头中，客户端收到这个响应后，将 cookie 保存在客户端（这里指的是浏览器的缓存中）。如下边图片的 set-cookie 字段就是后端返回的 cookie。
![image.png](https://cdn.nlark.com/yuque/0/2019/png/462392/1572416572758-3cc779bb-6bb6-453c-8866-fb47488edc57.png#align=left&display=inline&height=273&name=image.png&originHeight=273&originWidth=872&size=98323&status=done&width=872)

你可以在浏览器的 Application 里看到缓存的 cookie。
![image.png](https://cdn.nlark.com/yuque/0/2019/png/462392/1572416855193-791dcd77-50ff-4f18-8248-a4bb79c26a62.png#align=left&display=inline&height=163&name=image.png&originHeight=163&originWidth=818&size=18838&status=done&width=818)

当客户端再次发送同类请求时（同一资源路径），会携带保存在客户端的 cookie 信息，发到服务器，由服务器对会话跟踪。

cookie 是 web 开发技术   所有 web 语言都支持。
设置 cookie 路径：请求的时候只有对应的路径时候   才会携带此条 cookie
设置 cookie 有效时间：
大于 0  存放在客户端硬盘
小于 0  存放在浏览器的缓存中   默认
等于 0  一生成   就失效

```javascript
Cookie cookie = new Cookie(“key”,”value”)；
cookie.setMaxAge(60)； //设置cookie的生存期60秒
cookie.setPath(“/test”)；//设置cookie的路径
```

cookie 存放用户信息是很不安全的，别人很容易能获取到，浏览器也可以禁止使用 cookie ，但是很多网站禁止 cookie 后就不能访问了。

所以就有了 session 的出现（我是这样理解的）。
session  意思就是会话，  是 web 开发一种会话状态跟踪技术， cookie 也是。不同的说 session  保存在服务器。session 专门用于存放数据的集合，有读写的方法。主要是在服务端  
比如：在接口 a 时候存入 session  在接口 b 中可以获取存入的 sesion。

客户端访问一个接口   就是一个会话   服务器会为每一个会话维护一个 Session  就是 Session 对象

服务器对当前应用中的 Session 对象以 map 形式管理，这个 map 被叫做 session 列表。这个 map 的 key 是一个 32 为随机字符串   被叫做 JSESSIONID（java 里是这样，其他语言会有别的名称），value 为 session 对象。就像这样：
![image.png](https://cdn.nlark.com/yuque/0/2019/png/462392/1572343705712-413d4211-4a65-4a60-ad64-a476e1050cdc.png#align=left&display=inline&height=350&name=image.png&originHeight=350&originWidth=613&size=160566&status=done&width=613)

在 Session 信息写入 Session 列表后，系统会将 JSESSIONID 作为 name，32 位字符串作为 value，并以 cookie 形式放到响应头里面   并伴随响应   发送到客户端。

客户端接受到这个 cookie  会将其放到浏览器缓存中，  只要浏览器不关闭，缓存中的 cookie 就不会消失。
当用户第二次请求时，  会将缓存中的 cookie  伴随请求头信息   一块发送到服务器。
服务器从请求中获取 cookie  根据 cookie 中的 JSESSIONID  从 map 列表中读取到 session 对象   然后进行一些操作。

session  超时   默认是 30 分钟   即 30 分钟不操作此 session  就会失效。

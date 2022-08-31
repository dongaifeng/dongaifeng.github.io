---
title: docker学习笔记
urlname: mqom00
date: '2019-11-05 17:53:32 +0800'
tags: []
categories: []
---

## 概念

docker 本身是一个容器运行载体或者叫管理引擎。
image 把我们做的程序和配置，依赖打包成一个可运行的环境，这个包就叫 image。
容器 就是 image 生成的实例 相当于解压缩 image 这个包。
仓库 存放一堆镜像的地方 相当于 github。

## 安装

[官网](https://docs.docker.com/)

## hello word

#### 阿里云镜像加速

由于 docker hub 国外网站比较慢，所以用国内的镜像(类似 npm 的淘宝镜像)。

1. 注册阿里云账号，获取专属加速 链接。[链接](https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors)
1. 将获取的链接地址配置进 other_args="--registry-mirro=你的地址"

vim /etc/sysconfig/docke

3. 重启 service docker restart
4. 查看是否成功 ps [-ef] grep docker

#### docker run hello-world

run 后面跟一个镜像名 先在本地找镜像 没有后从远程拉取，并在容器中运行。

## 原理

![image.png](https://cdn.nlark.com/yuque/0/2020/png/462392/1577874219795-16058ed5-ca96-4185-b9af-e7f154453c42.png#align=left&display=inline&height=271&name=image.png&originHeight=541&originWidth=1300&size=735117&status=done&style=none&width=650)

## 常用命令

docker info 信息
docker --help

#### docker images 本地的镜像列表

![image.png](https://cdn.nlark.com/yuque/0/2020/png/462392/1577874745793-d7fb7203-b982-418a-953c-b37f7d75a446.png#align=left&display=inline&height=271&name=image.png&originHeight=542&originWidth=1206&size=270319&status=done&style=none&width=603)
![image.png](https://cdn.nlark.com/yuque/0/2020/png/462392/1577874964996-41dae3ca-cb51-4f53-a820-14e60d4146c6.png#align=left&display=inline&height=106&name=image.png&originHeight=211&originWidth=874&size=65212&status=done&style=none&width=437)

#### docker search 镜像名 搜索某个镜像

#### docker pull 镜像名   下载某个镜像

#### docker rmi 镜像名   删除镜像   docker rmi -f 镜像名 强制删除

### 容器命令

#### 新建并启动 docker run [options] image [command] [arg...]

![image.png](https://cdn.nlark.com/yuque/0/2020/png/462392/1577876179538-437df857-1fa6-411c-abf7-2cf1c80ff75b.png#align=left&display=inline&height=230&name=image.png&originHeight=460&originWidth=678&size=224389&status=done&style=none&width=339)

#### 列出当前所有正在运行的容器 docker ps [options]

![image.png](https://cdn.nlark.com/yuque/0/2020/png/462392/1577876600833-8dc41388-1e55-44a8-8857-133ed1f56e74.png#align=left&display=inline&height=140&name=image.png&originHeight=279&originWidth=511&size=108270&status=done&style=none&width=255.5)

#### 退出容器

- exit  停止容器并退出
- ctrl+P+Q 不停止容器退出

#### 启动容器 docker start 容器 ID 或者容器名

#### 重启容器 docker restart  容器 ID 或者容器名

#### 停止容器 docker stop  容器 ID 或者容器名

#### 强制容器 docker kill  容器 ID 或者容器名

#### 删除已经停止容器 docker rm  容器 ID 或者容器名

加参数 -f  强制删除

#### 查看日志 docker logs -f -t --tail  容器 ID

- -t  加入时间戳
- -f  跟随最新日志打印
- --tail  后面跟一个数字   显示最后多少条

#### 查看容器内运行的进程  docker top  容器 ID

#### 查看容器内部细节 docker inspect  容器 ID

#### 进入正在运行的容器并以命令行交互

- docker exec -it  容器 ID /bin/bash。进入容器   在容器内打开新的终端，可以启动新的进程。
- docker attach  容器 ID  重新进入 。并启动命令行，不会启动新进程

## 镜像

是一种联合文件系统   分层的   一层套一层的   只读的。当容器启动时，一个新的可写层被加载到镜像的顶部。
这一层通常被称作“容器层”，“容器层”之下的都叫“镜像层”。
容器  `(container)`  的定义和镜像  `(image)`  几乎一模一样，也是一层套一层，唯一区别在于容器的最上面那一层是可读可写的。

#### docker commit  提交容器副本   生成新的镜像

docker commit -m="描述信息" -a="作者"  容器 ID  要创建成的镜像名: [标签]
案例：

1. 从 HUb 下载 tomcat 运行：docker run -it -p 8080:8080 tomcat

-p  小写   表示：主机端口：docker 容器端口。
-P  大写   端口随机分配
-i  交互
-t  终端

2. docker commit -m="自己做的镜像" -a="abc"  容器 id abc/tomcat: v1.0

### 数据卷

可以做数据持久化   从主机的一个文件夹映射到 docker 容器里一个文件夹   改变一个   另一个也可改变。

#### docker run -it /宿主机绝对路径：/容器内目录   镜像名称

例子：docker run -p 8080:80 -v $PWD/www/:/usr/share/nginx/html/ -d nginx
docker run -it -v /宿主机绝对路径目录:/容器内目录:ro 镜像名      命令(带权限)
可在 Dockerfile 中使用 VOLUME 指令来给镜像添加一个或多个数据卷

#### 容器间传递共享(--volumes-from)

## DockerFile

Dockerfile 是用来构建 Docker 镜像的构建文件，是由一系列命令和参数构成的脚本。

#### 基础知识

- 保留字指令必须大写   后面必须跟至少一个参数
- 从上到下执行
- #表示注释
- 每条指令都会创建一个新的镜像，并对镜像 commit

#### 执行 DockerFile 流程

1. docker 从基础镜像运行一个容器
1. 执行一条指令并对容器做出修改
1. 执行类似 docker commit 的操作   提交一个新的镜像层
1. docker 再基于刚才提交的镜像运行一个新的容器
1. 执行 DockerFile 的下一条命令

#### DockerFile 的内容

- FROM  基础镜像，当前新镜像是基于哪个镜像的
- MAINTAINER    镜像维护者的姓名和邮箱地址
- RUN   容器构建时需要运行的命令
- EXPOSE   当前容器对外暴露出的端口
- WORKDIR     指定在创建容器后，终端默认登陆进来工作目录，一个落脚点
- ENV    用来在构建镜像过程中设置环境变量
- ADD   将宿主机目录下的文件拷贝进镜像且 ADD 命令会自动处理 URL 和解压 tar 压缩包
- COPY   类似 ADD，拷贝文件和目录到镜像中.将从构建上下文目录中 <源路径> 的文件/目录复制到新的一层的镜像内的 <目标路径> 位置   COPY src dest
- VOLUME     容器数据卷，用于数据保存和持久化工作
- CMD   指定一个容器启动时要运行的命令.Dockerfile 中可以有多个 CMD 指令，但只有最后一个生效，CMD 会被 docker run 之后的参数替换
- ENTRYPOINT    指定一个容器启动时要运行的命令.ENTRYPOINT 的目的和 CMD 一样，都是在指定容器启动程序及参数
- ONBUILD    当构建一个被继承的 Dockerfile 时运行命令，父镜像在被子继承后父镜像的 onbuild 被触发

### 自定义镜像

base 镜像(scratch)  基础镜像   大部分镜像都是就从他上边构建的。

#### 构建命令： docker build -t  新镜像名字: tag   

例子：docker build -t nginx:ai -f Dockerfile .
-f  是指定 dockerfile 文件
-t  是指定镜像名称
.  表示当前目录

#### 列出镜像历史： docker history  镜像名

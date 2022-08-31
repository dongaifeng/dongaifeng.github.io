---
title: Docker Compose
urlname: blfsqn
date: '2020-01-20 16:20:15 +0800'
tags: []
categories: []
---

通过一个 docker-compose.yml 文件管理多个容器。

## 安装

官网下载，安装教程
[https://docs.docker.com/compose/install/](https://docs.docker.com/compose/install/)

#### linux  安装

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.25.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

输入命令  ll  发现 docker-compose 没有执行权限
输入一下命令   设置可执行

```bash
sudo chmod +x /usr/local/bin/docker-compose
```

#### 检验安装是否成功

docker-compose version

# 常用命令

#### docker-compose up -d    构建、启动容器

- -d  是守护进程方式

#### docker-compose down

#### docker-compose kill myserve     通过发送 SIGKILL 信号来停止指定服务的容器

#### docker-compose scale myserve=3    设置指定服务运气容器的个数，以 service=num 形式指定

#### docker rm myserve  删除指定服务的容器

#### docker start myserve 启动指定服务已存在的容器

#### docker build  构建或者重新构建服务

#### docker-compose stop myserve    停止已运行的服务的容器

#### docker-compose run web bash    在一个服务上执行一个命令

#### docker logs  查看日志

原样粘贴   vim 打开文件以后   输入 :set paste

# YAML  配置文件

YAML 专门用来写配置文件的，类似 json。

#### 特性

- 大小写敏感
- 使用缩进表示层级关系
- 缩进不允许用 tab，只能用空格
- 缩进空格数目不重要，只要同层级元素左侧对齐，能够展现层级关系
- #表示注释

#### YAML 对象

对象是一组键值对，用冒号表示结构
object: myobject

#### YAML  数组

一组以中划线开头的行，构成一个数组

```bash
- Cat
- Dog
- Fish
```

如果数组的某一元素是数组

```bash
- Array
  - Cat
  - Dog
```

# docker-compose.yml 属性

- version  指定 docker-compose.yml 文件的写法格式
- services  容器的集合
- build  配置构建时，Compose 会利用它自动构建镜像，该值可以是一个路径，也可以是一个对象，用于指定 Dockerfile 参数
- command  覆盖容器启动后默认执行的命令
- environment  环境变量配置，可以用数组或字典两种方式
- expose   暴露端口，只将端口暴露给连接的服务，而不暴露给主机
- volumes  数据卷

# 部署 MySQL

在/usr/local/mydocker/mysql  目录下新建 docker-compose.yml 文件

```bash
  db:
    image: mysql # 使用这个镜像去构建
    restart: always #表示总是重启
    environment:
      MYSQL_ROOT_PASSWORD: 123456 # 初始化密码
    command: #这些是myaql的一些变量
      --default-authentication-plugin=mysql_native_password
      --character-set-server=utf8mb4
      --explicit_defaults_for_timestamp=true
      --lower_case_table_names=1
    ports:
      - 3305:3306 // 端口号 左边是linux主机暴露的端口号，右边是容器的端口号
    volumes:
      - ./data:/var/lib/mysql  # 数据卷 冒号左边是linux主机目录，右边是容器目录

# 一个页面版的数据谷工具
  adminer:
    image: adminer
    restart: always
    ports:
      - 7777:8080
```

运行  docker-compose up  运行

# 部署 gitLab

```bash
gitlab:
  image: 'twang2218/gitlab-ce-zh:11.1.4'
  restart: unless-stopped
  hostname: '47.105.185.203'
  environment:
    TZ: 'Asia/Shanghai'
    GITLAB_OMNIBUS_CONFIG: |
      external_url 'http://47.105.185.203'
      gitlab_rails['time_zone'] = 'Asia/Shanghai'
      gitlab_rails['gitlab_shell_port'] = 2222
      unicorn['port'] = 8888
      nginx['listen_port'] = 80
  ports:
    - '8081:80'
    - '443:443'
    - '2222:22'
  volumes:
    - config:/etc/gitlab
    - data:/var/opt/gitlab
    - logs:/var/log/gitlab
```

更多配置可以查看 dockerHub 上的镜像[gitlab](https://hub.docker.com/r/twang2218/gitlab-ce-zh)

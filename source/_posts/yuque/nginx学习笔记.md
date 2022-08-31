---
title: nginx学习笔记
urlname: qibhr1
date: '2019-11-09 21:33:57 +0800'
tags: []
categories: []
---

## 1 安装

### Linux 系统下安装

[nginx-1.12.2.zip](https://www.yuque.com/attachments/yuque/0/2020/zip/462392/1601282900249-1414edc4-07fe-4ce9-b588-72dde4b8ca44.zip?_lake_card=%7B%22uid%22%3A%221601282898887-0%22%2C%22src%22%3A%22https%3A%2F%2Fwww.yuque.com%2Fattachments%2Fyuque%2F0%2F2020%2Fzip%2F462392%2F1601282900249-1414edc4-07fe-4ce9-b588-72dde4b8ca44.zip%22%2C%22name%22%3A%22nginx-1.12.2.zip%22%2C%22size%22%3A1449245%2C%22type%22%3A%22application%2Fx-zip-compressed%22%2C%22ext%22%3A%22zip%22%2C%22progress%22%3A%7B%22percent%22%3A99%7D%2C%22status%22%3A%22done%22%2C%22percent%22%3A0%2C%22id%22%3A%22wlYLL%22%2C%22card%22%3A%22file%22%7D) win 版本
安装 nginx 需要先安装几个插件
yum install gcc openssl openssl-devel pcre pcre-devel zlib zlib-devel -y

## 2 启动

### 普通启动

切换到 nginx 的 sbin 目录下 执行： ./nginx

### 实际工作中 通过输入路径 启动

/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf

### 查看启动进程

ps -ef|grep nginx
nginx 有两个进程 master 进程读取配置文件，并维护 woker 进程。
woker 进程对请求进行实际操作。

### 3 关闭 Nginx

### 优雅关闭

找出主进程 master 号 ps -ef|grep nginx
kill QUIT 主进程 ID  
优雅关闭会在处理完已进来的请求并返回数据后才关闭

### 快速关闭

kill -TERM 主进程 ID
快速关闭 就是直接关闭 不在执行任务

#### 重启 nginx 

./nginx -s reload

### 检查修改配置文件是否正确

/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf -t

## 4 window 安装

根目录下 启动：start nginx
关闭： nginx -s stop

## 5 配置文件 nginx.conf

```nginx
#======基本配置
#user  nobody;  #配置woker进程运行用户 nobody默认用户 或者用root
worker_processes  1; #配置工作进程数目 cpu的两倍

#error_log  logs/error.log;#配置错误日志及类型 debug/info/notice/ error是默认的
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;# 进程pid文件 .remp文件

# ========配置工作模式连接数
events {
    #use epoll  #用epoll事件模型 优化用的 一般不用
    worker_connections  1024; #配置woker进程最多可连接的数 总连接数 = 这个数 * worker_processes
}

#=============http配置
http {
  #--------------基本配置
    include       mime.types; #配置nginx支持的多媒体文件 值是个文件 里面又文件可是列表
    default_type  application/octet-stream; #默认文件类型 流类型

  #配置日志格式 $...是nginx的内部变量
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

  # 配置access.log 的存在那个路径  设置使用上边定义的main格式
    #access_log  logs/access.log  main;

    sendfile        on; # 开启搞笑文件传输模式  优化用
    #tcp_nopush     on; # 防止网络阻塞

    #keepalive_timeout  0;
    keepalive_timeout  65; # 长连接超时时间 秒

    #gzip  on; #gzip压缩开始
#------------server配置
    server {
        listen       80; #监听的端口
        server_name  localhost; # 配置服务名称

        #charset koi8-r; # 字符集

        #access_log  logs/host.access.log  main; # 和上面的那个一样
#默认 匹配 / 的请求 当访问路径中有 / 会被locatin 匹配到
        location / {
            root   html; # 配置网站默认的根目录 这里是nginx下的html 为根目录
            index  index.html index.htm; # 首页名称
        }

        #error_page  404              /404.html; # 配置404页面 还可以配置500

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
    # = 是精确匹配
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}
# 禁止访问 .htaccess文件
        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server  配置 https 服务 安全 加密  默认端口443
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem; #需要购买证书 wodign
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}

```

## 反向代理

###   设置 linux 对外开放端口

firewall-cmd --aa-port=8888/tcp --permanent  对外开放 8888 端口
 firewall-cmd-reload 重启
 firewall-cmd-list-all 查看

```nginx
  server {
        listen       1111;
        server_name  192.168.43.33;

        #charset koi8-r;
        #access_log  logs/host.access.log  main;

        location / {
            root   html;
    #访问nginx的192.168.43.33：1111 转发到 http://127.0.0.1:3000
            proxy_pass  http://127.0.0.1:3000;
            index  index.html index.htm;
        }
```

---
title: linux学习笔记
urlname: mgcx72
date: '2019-11-05 17:54:14 +0800'
tags: []
categories: []
---

## 文件目录

![image.png](https://cdn.nlark.com/yuque/0/2019/png/462392/1572947759021-110f5a05-6387-4a99-9b77-59dda89730e2.png#align=left&display=inline&height=252&margin=%5Bobject%20Object%5D&name=image.png&originHeight=252&originWidth=523&size=38225&status=done&style=none&width=523)

**/bin  存放指令,二进制可执行文件**
/sbin  存放系统管理员   使用的指令
/root  系统管理员   超级权限者目录
/boot  启动 linux 时使用的   核心文件
**/var  存放不断扩充的东西   比如日志**
/dev  设备管理器   硬件用文件形式存储
/mnt 默认挂载别的系统的文件   外部文件挂载/mnt/上
**/etc 存放配置的相关文件**
/proc  虚拟目录  
/srv  /proc   /sys  服务启动文件   等系统目录   别动
/media U 盘   光盘出现的地方
/opt  安装软件要放在这
**/usr 安装一个软件的默认目录，相当于 windows 下的 program files**
**/usr/local    软件安装完成放在这里**

#### 软件安装目录

比如 mysql 安装会把配置文件装到/etc/mysql 目录下。数据会被放到/var/mysql 目录下。可执行文件会被放到/bin /mysql 目录下

# 基本指令

### 系统管理

shutdown -h now（立即进行关机）
shutdown -r now （现在重新启动计算机）      
reboot （现在重新启动计算机）
startx  进入桌面

### 操作目录，文件

#### pwd  显示路径

#### cd  切换到指定目录    

例子：
cd /root  相对路径      
cd ../../root  绝对路径  
cd ..  返回上一级

#### ls  显示文件列表

参数： 
-l  列出文件详细信息
-a  李处所有文件

#### mkdir  创建目录

语法：mkdir 【选项】要创建目录      
例子：mkdir  /home/dog
参数：-p  创建多级目录  
例子 mkdir -p /home/dog/two

#### rmdir  删除目录

语法：rmdir  要删除目录
例子：rmdir  /home/dog    删除 dog 这个文件夹，  文件夹下不能有文件
参数 -rf   强制删除
例子：rmdir -rf /home/dog

#### rm  删除文件或目录

参数：
-f  强制删除
-r  删除该目录下所有文件

#### mv  移动文件，目录

#### find  在文件系统中查找指定的文件

#### grep  在指定的文件夹中查找指定的字符串

#### more  分页显示

#### touch  创建文件

例子：touch hello.txt    创建一个空文件

#### cp  拷贝

参数 -r  递归拷贝
例子：cp a.txt bbb/  将当前目录下的 a.txt  拷贝到当前目录的 bbb 这个目录下
例子：cp -r aaa/  bbb/    将 home/aaa 文件夹的（包括 aaa） 拷贝到 /home/bbb 文件夹里

#### cat  查看文件内容

#### echo  生成一个带内容的文件

例子：
echo aaa > aaa.txt     >追加
echo bbb >> bbb.txt     >>  覆盖

### 进程管理

在 linux 中每个执行程序都是一个进程，都有一个进程 ID，每个进程都有父进程。
进程有前台和后台之分。

#### ps  查看进程

例子： ps -aux | grep xxx    竖线是管道   表示先查看进程，然后再查找 xxx 的进程
例子： ps -aux | grep sshd   查看 sshd 进程
参数：

- -a 显示当前终端所有进程信息
- -u 以用户的格式显示进程信息
- -x 显示后台进程运行的参数

#### ps 显示内容详解

![image.png](https://cdn.nlark.com/yuque/0/2020/png/462392/1578818683111-8d7c672a-52c3-439b-8815-8abdfeab12e1.png#align=left&display=inline&height=289&margin=%5Bobject%20Object%5D&name=image.png&originHeight=470&originWidth=813&size=129870&status=done&style=none&width=500)

#### 常用方式： ps -ef | more

- -e 显示所有进程
- -f 全格式
- UID 是用户 ID，PID 是进程 ID，PPID 父进程 ID

#### kill 终止进程

kill [选项] 进程 id   -9：强迫进程立即停止
kill 3305
kill -9 5050
killall 进程名称
killall nginx

#### pstree  查看进程树

语法： pstree [选项] PID 
选项：

- -p 显示进程 PID
- -u 进程所属用户

例子： pstree -p

#### tar  压缩

语法：tar [-cxzjvf]  压缩打包文档的名称   想要打包的目录
参数：

- -c  建立一个归档文件的参数指令
- -x  解开归档文件
- -z  是否使用 gzip 压缩
- -j  是否使用 bzip2  压缩
- -v  过程显示文件
- -f  使用档名   在 f 之后跟档名
- -tf  查看归档文件里面的信息

例子：
压缩文件夹  tar -zcvf test.tar.gz test\
解压缩  tar -zxvf text.tar.gz

### rpm 包和 yum

rpm 是一种互联网下载包的管理工具，类似 npm，yum 是 rpm 的高级版

#### 查看已安装 rpm 包 : rpm -qa|grep xxx

例子 rpm -qa | grep nginx

#### 查询包名称

rpm -qi nginx
查询包安装到哪里： rpm -ql firefox 
删除：rpm -e nginx
安装： rpm -ivh rpm 包全称 i=install v=提示信息 h=进度条

#### yum 指令

yum list | grep xx 包
yum install xx 包

---
title: Linux 手动部署 war 包
toc: true
layout: blog
categories:
  - Blog
  - Linux
tags: 
  - Linux
  - War
date: 2019-02-28 11:00:13
---
执行 `shutdown.sh` 时关闭 `Tomcat` 与 进程,
<!-- more -->

# XShell 连接	

* 安装 lrzsz		

` yum -y install lrzsz `	

* 上传 `Tomcat` 到 `linux`	

> 下载 `Linux` 版本 `tomcat` 

进入官网<a href="https://tomcat.apache.org/" target="_blank">Tomcat</a>，下载历史版本选择则 Download 中的 Archives选项
```
https://archive.apache.org/dist/tomcat/tomcat-8/v8.0.52/bin/
```
**注：** 文件夹为 `bin` 而不是 `src`

> 上传文件到 `linux` 当前目录下	

```text
# rz 打开文件对话框(出现错误)

# rz -be 能够正确上传
```

* 解压 `tomcat` 文件	

```shell
tar zxvf apache-tomcat-8.0.52.tar.gz 
```

> 解压`xx.tar`文件	

```sh
tar xvf xx.tar
```

```
进入../bin 目录，执行该命令
chmod u+x *.sh
```
> 清空`catalina.out`日志文件	

引用<a href="https://www.cnblogs.com/ainihaozhen/p/9466524.html" target="_blank">linux清空 catalina.out 日志 不需要重启 tomcat</a>

```sh
[root@CDH46 logs]# du -h catalina.out  # 查看日志文件大小
17M	catalina.out 					   
[root@CDH46 logs]# >catalina.out  	   # 重定向清空文件
```

> 分配 `JVM` 内存空间，记录 `GC` 日志	

```sh
编辑 catalina.sh 文件,增加
# create gc log monitor
# OS specific support.  $var _must_ be set to either true or false.
JAVA_OPTS='-Xms2048m -Xmx2048m -XX:PermSize=256m -XX:MaxNewSize=512m -XX:MaxPermSize=512m -XX:ParallelGCThreads=8 -XX:+UseConcMarkSweepGC -Xloggc:/opt/apache-tomcat-8.0.52/logs/tomcat_gc.log'

# OS specific support.  $var _must_ be set to either true or false.
JAVA_OPTS='-Xms2048m -Xmx2048m -XX:PermSize=256m -XX:MaxNewSize=512m -XX:MaxPermSize=512m -XX:ParallelGCThreads=8 -XX:+UseConcMarkSweepGC'
```

> 指定`tomcat`依赖的 `jdk` 版本	

1. 找到 `catalina.sh` 与 `setclasspath.sh` 文件路径	

```sh
[root@cdh02 bin]# pwd
/opt/iptvOrder/apache-tomcat-8.0.52/bin
[root@cdh02 bin]# vi ./catalina.sh 
```

2. 在`catalina.sh`文件头部加入以下配置,指定 `jdk` 及 `jre` 路径

```sh
# specify jdk version
JAVA_HOME=/usr/java/jdk1.8.0_101
JRE_HOME=/usr/java/jdk1.8.0_101/jre
```

> shutdown 命令 kill 进程 

1. 修改`catalina.sh`文件

```sh
export JAVA_HOME=/usr/java/jdk1.8.0_101
export JRE_HOME=/usr/java/jdk1.8.0_101/jre
export CATALINA_HOME=/opt/apache-tomcat-8.0.52
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$JAVA_HOME/bin:$PATH
```

* 需要注意`JAVA_HOME、JRE_HOME、CATALINA_HOME`路径配置

2. 修改`catalina.sh`文件,设置记录CATALINA_PID(catalina.sh)

```sh
# Get standard environment variables
PRGDIR=`dirname "$PRG"`
if [ -z "$CATALINA_PID" ]; then
  CATALINA_PID=$PRGDIR/CATALINA_PID
fi
```

**注:** 

* 设置会在启动时候bin下新建一个CATALINA_PID文件
* 关闭时候从CATALINA_PID文件找到pid，kill。。。同时删除CATALINA_PID文件

3. 修改`shutdown.sh`

```sh
# 这是 8.0.50 版本最后一行
exec "$PRGDIR"/"$EXECUTABLE" stop -force "$e@"

# 这是 8.5.38 版本最后一行
exec "$PRGDIR"/"$EXECUTABLE" stop -force "$@"
```

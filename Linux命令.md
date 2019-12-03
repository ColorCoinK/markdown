---
title: Linux 使用记录
toc: true
layout: blog
categories:
  - Blog
  - Linux
tags:
  - Linux
date: 2019-02-28 11:00:13
---
{{title}}
<!-- more -->

# `Linux` 命令使用记录

## 查看已经在使用的端口	

```linux
netstat -nultp
```

## 检查端口是否被占用	

```linux
netstat -anp|grep 7090
```

## 删除文件夹	

```linux
rm -rf xx
```

## 删除文件	

```linux
rm -f xx.log
```

## 查看 `tomcat` 相关的进程	

```linux
ps -ef|grep tomcat
```

## 查看 `tomcat` 实时输出日志		

```linux
tail -f /opt/tomcat/logs/catalina.out
```

## 查看内存信息	

<a href="https://blog.csdn.net/zhangliao613/article/details/79021606" target="_blank">参考地址</a>

```
[root@CentOS7 apache-tomcat-8.0.52]# cat /proc/cpuinfo | grep 'physical id' | uniq	-- cpu 个数
physical id	: 0
physical id	: 2
[root@CentOS7 apache-tomcat-8.0.52]# cat /proc/cpuinfo | grep 'cpu cores' | uniq	 	-- cpu 核数
cpu cores	: 1
[root@CentOS7 apache-tomcat-8.0.52]# cat /proc/cpuinfo | grep 'model name' | uniq		-- cpu 型号
model name	: Intel(R) Xeon(R) CPU E7-4809 v2 @ 1.90GHz
[root@CentOS7 apache-tomcat-8.0.52]# 
```

## 某个进程 CPU 占用	

```linux
top -p 8104
```

## `Linux` 下 `Tomcat` 开启查看 `GC` 信息	

> 1. 在 tomcat 的安装目录下,找到 `bin/catalina.sh`文件	

* 修改前：

```
JAVA_OPTS='-Xms2048m -Xmx2048m -XX:PermSize=256m -XX:MaxNewSize=512m -XX:MaxPermSize=512m'
```

* 修改后：

```shell 
# create gc log monitor
# OS specific support.  $var _must_ be set to either true or false.
JAVA_OPTS='-Xms2048m -Xmx2048m -XX:PermSize=256m -XX:MaxNewSize=512m -XX:MaxPermSize=512m -XX:ParallelGCThreads=8 -XX:+UseConcMarkSweepGC -Xloggc:/opt/apache-tomcat-8.0.52/logs/tomcat_gc.log'

# OS specific support.  $var _must_ be set to either true or false.
JAVA_OPTS='-Xms2048m -Xmx2048m -XX:PermSize=256m -XX:MaxNewSize=512m -XX:MaxPermSize=512m -XX:ParallelGCThreads=8 -XX:+UseConcMarkSweepGC'
```

## `Linux` 下查看 `Tomcat` 并发数

```
netstat -anp|grep 7090|grep ESTABLISHED -c
```
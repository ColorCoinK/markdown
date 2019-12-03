---
title: Linux安装基本工具
toc: true
layout: blog
categories:
  - Blog
  - Linux
tags:
  - Blog
  - Linux
date: 2019-04-12 13:55:25
---

{{ title }}

<!-- more -->

# Linux初始化安装必要程序

## `CentOS` 安装`yum`

1. 下载最新的 <a href="http://yum.baseurl.org/download/" target="_blank">`yum-3.4.3`</a>解压并下载

```
# 下载
wget http://yum.baseurl.org/download/3.4/yum-3.4.3.tar.gz

# 解压命令
tar xvf yum-3.4.3.tar.gz
```

2. 进入目录,运行安装  

```
cd yum/yum-3.4.3

yummain.py install yum
```

* 如果结果提示错误:`CRITICAL: yum.cli:Config Error accessing file for config file:///etc/`,可能是缺少配置文件。在`etc`目录下新建`yum.conf`文件,然后再次运行`yummain.py install yum`,顺利完成安装。

3. 更新系统

```
yum check-update

yum update

yum clean all
```

## 安装`lrzsz`使用`rz、sz`命令

> 安装`lrzsz`

```
yum -y install lrzsz
```

> 上传

```
rz 
```

* 上传文件较大或出现错误时,使用`rz -be`命令
* 上传并替换`rz -y `

> 下载

```
sz
```

* 上传文件较大或出现错误时,使用`sz -be`命令

## `Tomcat` 关闭时 `Kill` 进程

可以参照另一篇记录<a href="../Linux/Linux部署war包/" target="_blank">Linux部署war包</a>
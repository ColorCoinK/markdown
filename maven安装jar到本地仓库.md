---
title: maven安装jar到本地仓库
toc: true
layout: blog
categories:
  - Blog
tags:
  - Blog
  - maven
date: 2019-08-16 11:12:27
---
在使用Maven开发过程中,难免会遇到jar包无法从中央仓库下载的窘境。本博客介绍一种方法,也许能解决遇到的问题。
<!-- more -->

# 使用`Maven`将`pom.xml`中下载失败的`jar`安装到本地仓库或私有仓库

* 以`impalaJDBC41`文件为例

> 下载相应的`jar`文件

* 原先只需要在maven中增加对`ImpalaJDBC41`的依赖配置即可

```xml
<properties>
  <impala.version>2.6.3</impala.version>
</properties>

<dependencies>
  <!-- 有可能下载不到该Jar, 可以到该地址下载相应版本：https://www.cloudera.com/downloads/connectors/impala/jdbc/2-6-3.html -->
  <!-- https://mvnrepository.com/artifact/com.cloudera/ImpalaJDBC41 -->
  <dependency>
    <groupId>com.cloudera</groupId>
    <artifactId>ImpalaJDBC41</artifactId>
    <version>${impala.version}</version>
  </dependency>
<dependencies>
```

* 不知为何,配置的`阿里云镜像库`无法下载该`jar`

改为从<a href="https://www.cloudera.com/downloads/connectors/impala/jdbc/2-6-3.html" target="_blank">下载相应的版本及`jar`文件</a>

> 打开Dos窗口,使用如下命令

```dos
mvn install:install-file -DgroupId=com.cloudera -DartifactId=ImpalaJDBC41 -Dversion=2.6.3 -Dpackaging=jar -Dfile=./ImpalaJDBC41-2.6.3.jar
```

- `DgroupId`: `pom.xml`配置中`groupId`的值
- `DartifactId`: `pom.xml`配置中`artifactId`的值
- `Dversion`: 版本号
- `Dpackaging`: 文件类型
- `Dfile`: 文件路径


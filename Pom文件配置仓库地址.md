---
title: 配置单个 maven 项目 pom 依赖仓库地址
toc: true
layout: blog
categories:
  - Blog
  - Maven
tags:
  - Maven
date: 2019-02-28 11:00:13
---
{{title}}
<!-- more -->
# `pom.xml` 配置 `maven` 仓库地址，不需要改 `setting`	

> `pom.xml`文件中加入`repositories`配置	

```xml
<!--配置maven阿里云仓库开始,不用去改maven的setting -->
<repositories>
	<repository>
		<id>public</id>
		<name>local private nexus</name>
		<url>http://maven.aliyun.com/nexus/content/groups/public/</url>
		<releases>
			<enabled>true</enabled>
		</releases>
	</repository>
</repositories>
<pluginRepositories>
	<pluginRepository>
		<id>public</id>
		<name>local private nexus</name>
		<url>http://maven.aliyun.com/nexus/content/groups/public/</url>
		<releases>
			<enabled>true</enabled>
		</releases>
		<snapshots>
			<enabled>false</enabled>
		</snapshots>
	</pluginRepository>
</pluginRepositories>
<!--配置maven阿里云结束 -->
```
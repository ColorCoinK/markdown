---
title: Eureka-Server
toc: true
layout: springcloud
categories:
  - SpringCloud
tags:
  - SpringCloud
  - Java
date: 2019-02-27 15:20:13
---
{{title}}
<!-- more -->

# 基于 Eureka 搭建高可用(双节点服务注册中心)  

本项目是一个基于 SpringBoot(1.5.19.RELEASE)、SpringCloud(Dalston.SR5)、Eureka 的服务注册中心

- 配置文件说明 

> `application.yml`: 

  单个 `Eureka` 服务注册中心

> `application-colony.yml`: 

  双节点注册中心示例,实现双向注册从而提高可用性

1. 默认使用 `application.yml` 配置文件;
2. 使用`application-conlony.yml` 配置文件时,需要指定激活的配置项.(例如启动`server-1`,需运行`java -jar xx.jar --spring.profiles.active=server-1`)
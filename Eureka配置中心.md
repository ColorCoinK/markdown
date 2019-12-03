---
title: Eureka配置中心
toc: true
layout: springcloud
type: springcloud
categories:
  - SpringCloud
tags:
  - SpringCloud
  - Java
date: 2019-04-03 13:25:05
---
{{title}}
<!-- more -->

# `SpringCloud` 搭建配置中心`SpringCloud config` 

## 添加依赖 

```xml
    <!-- eureka client start-->
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <!-- eureka client end-->

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- config server start -->
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-config-server</artifactId>
    </dependency>
    <!-- config server end -->
```

**注:** 

  - `JDK version`: 1.8
  - `SpringBoot version`:  `2.1.2.RELEASE`
  - `SpringCloud version`: `Greenwich.RELEASE`

`SpringCloud` 版本兼容 <a href="https://spring.io/projects/spring-cloud" target="_blank">`https://spring.io/projects/spring-cloud`</a>

## 配置`application.yml`

```yaml
server:
  port: 3001

spring:
  application:
    name: microservice-config-server
  # Git configuration
  cloud:
    config:
      server:
        git:
          uri: https://github.com/forezp/SpringcloudConfig/
          search-paths: respo
          default-label: master
#          uri: https://git.oschina.net/itmuch/spring-cloud-config-repo.git # Git 仓库地址
#          username:  # Git 仓库账号
#          password:  # Git 仓库密码
```

  > `yml`
  
  - `spring.cloud.config.server.git.uri`: 配置`GitHub`仓库地址(`Gitee`地址也可以)
  - `spring.cloud.config.server.git.search-paths`: 配置仓库路径
  - `spring.cloud.config.label`: 配置仓库的分支
  - `spring.cloud.config.server.git.username`: 访问`GitHub`仓库的用户名
  - `spring.cloud.config.server.git.password`: 访问`GitHub`仓库的用户密码

  > 路径规则  
  
  `Spring Cloud Config Server`提供了`RESTful API`,可用来访问存在`Git`仓库中的配置文件.
  ```yaml
    /{application}/{profile}[/{label}]
    /{application}-{profile}.yml
    /{label}/{application}-{profile}.yml
    /{application}-{profile}.properties
    /{label}/{application}-{profile}.properties
  ```
  其中的{appliation}、{profile}、{label} 都是占位符.
  

**注**

  - 使用的不同配置仓库,可以更改为自己的`GitHub`仓库地址
  - 公开仓库不需要账号、密码,私有仓库需要填写(本`Demo`为公开仓库)。
  
## 测试 

访问地址: <a href="http://127.0.0.1:3001/foo/dev" target="_blank">`http://127.0.0.1:3001/foo/dev`</a>

## 配套代码

* GitHub: https://github.com/ColorCoinK/springcloud-learning/tree/master/springcloud-config-server
  
**TIPS** 

  - 注册到`Eureka Server`时,需要添加在 `application.yml` 添加 `eureka.client.service-url.defaultZone=http://localhost:1001/eureka/`,同时需要在`SpringcloudConfigServerApplication`(启动类)添加`@EnableEurekaClient`注解
  - 参考博客 
    1. <a href="http://www.itmuch.com/spring-cloud/finchley-19/" target="_blank">Spring Cloud Config —— 周立(`http://www.itmuch.com/spring-cloud/finchley-19/`)</a>
    2. <a href="https://blog.csdn.net/forezp/article/details/81041028" target="_blank">Spring Cloud Config —— 方志朋(`https://blog.csdn.net/forezp/article/details/81041028`)</a>
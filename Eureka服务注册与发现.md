---
title: Eureka服务注册中心
toc: true
layout: springcloud
type: springcloud
categories:
  - SpringCloud
tags:
  - SpringCloud
  - Java
  - Eureka
date: 2019-04-03 10:45:50
---
搭建Eureka(Edgware.SR5版本)服务注册中心
<!-- more -->

# 基于 Eureka 搭建高可用(双节点服务注册中心)

> 添加`pom`依赖

```xml
<parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-parent</artifactId>
  <version>1.5.19.RELEASE</version>
  <relativePath/> <!-- lookup parent from repository -->
</parent>

<properties>
  <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
  <java.version>1.8</java.version>
  <spring-cloud.version>Edgware.SR5</spring-cloud.version>
</properties>

<dependencies>
  <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter</artifactId>
  </dependency>

  <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka-server</artifactId>
  </dependency>

  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
  </dependency>
</dependencies>

<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-dependencies</artifactId>
      <version>${spring-cloud.version}</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>

<build>
  <plugins>
    <plugin>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-maven-plugin</artifactId>
    </plugin>
  </plugins>
</build>
```

> 版本说明 

* 本项目是一个基于 SpringBoot(1.5.19.RELEASE)、SpringCloud(Edgware.SR5)、Eureka 的服务注册中心

## 启动类 @EnableEurekaServer

```java
@EnableEurekaServer
@SpringBootApplication
public class EurekaServerApplication {

	public static void main(String[] args) {
		SpringApplication.run(EurekaServerApplication.class, args);
	}

}
```

## 配置文件

> `application.yml` 
  
单个 `Eureka` 服务注册中心

> `application-colony.yml`  
  
双节点注册中心示例,实现双向注册从而提高可用性详细可以看<a href="http://www.ityouknow.com/springcloud/2017/05/10/springcloud-eureka.html">注册中心Eureka</a>

1. 默认使用 `application.yml` 配置文件;

2. 使用`application-conlony.yml` 配置文件时,需要指定激活的配置项.(例如启动`server-1`,需运行`java -jar xx.jar --spring.profiles.active=server-1`)

## 配套代码

GitHub : <a href="https://github.com/ColorCoinK/springcloud-learning/tree/master/eureka-server" target="_blank">服务注册中心Eureka-Server</a>

> 推荐阅读

* 翟永超: <a href="http://blog.didispace.com/springcloud1/" target="_blank">Eureka服务注册与发现</a>
* 周立: <a href="http://www.itmuch.com/spring-cloud/finchley-5/" target="_blank">SpringCloud服务注册与发现Eureka</a>

> 相关文章

<a href="http://www.itmuch.com/spring-cloud/finchley-6/">周立——Eureka服务注册与发现深入</a>

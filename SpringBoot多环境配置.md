---
title: SpringBoot 多环境配置	
toc: true
layout: blog
categories:
  - SpringBoot
tags:
  - SpringBoot
  - properties
date: 2019-02-28 11:00:13
---
{{title}}
<!-- more -->

# <h3>SpringBoot 多环境配置</h3>

在开发环境、生产环境、测试环境中使用的配置也许有些不同，使用同一个配置文件时每次`package`前都需要修改成指定的环境配置。写该笔记已解决上述困，坑又被填平了一个....真好。	

**当前开发环境:** `SpringBoot`	
**适用场景:** 多环境指定配置参数	

> 划重点

1. 命名规范`application-xx.properties`或`appliction-xx.yml`
2. 使用当前配置构建的 `jar` 文件包含多环境的配置文件，可以通过`java -jar xx.jar  --spring.profiles.active=prod`命令加载指定的配置文件

## 多配制文件(推荐)	

**主文件**	

`application.yml`
```properties
spring:
  profiles:
    active: dev #指定应用`pacakage`时加载的文件名
  mvc:
    static-path-pattern: /**
    view:
      prefix: /WEB-INF/jsp/
      suffix: .jsp
  resources:
    static-locations:
    -  classpath:/static,classpath:/templates,file:${web.upload-path}
```

**开发环境**		

`application-dev.yml`	
```properties
server:
  port: 8090
spring:
  datasource:
    driver-class-name: oracle.jdbc.driver.OracleDriver
    url: jdbc:oracle:thin:@192.168.203.158:1521/test
    username: test
    password: test
  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: true
```

**生产环境**	

`application-prod.yml`	
```properties
server:
  port: 8090
spring:
  datasource:
    driver-class-name: oracle.jdbc.driver.OracleDriver
    url: jdbc:oracle:thin:@127.0.0.1:1521/test
    username: 数据库用户名
    password: 数据库密码
  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: true
```

## 单配置文件 

> `application-conlony.yml` 

```yaml
server:
  port: 1001
spring:
  profiles: server-1
eureka:
  instance:
    hostname: server-1
  client:
    register-with-eureka: false
    fetch-registry: false
    service-url:
#      defaultZone: http://${eureka.instance.hostname}:${server.port}/${spring.application.name}
      defaultZone: http://server-2:1002/eureka-server

---

  server:
    port: 1002
  spring:
    profiles: server-2
  eureka:
    instance:
      hostname: server-2
    client:
      register-with-eureka: false
      fetch-registry: false
      service-url:
        #      defaultZone: http://${eureka.instance.hostname}:${server.port}/${spring.application.name}
        defaultZone: http://server-1:1001/eureka-server
```

> 启动  

```yaml
使用`application-conlony.yml` 配置文件时,需要指定激活的配置项.(例如启动`server-1`,需运行`java -jar xx.jar --spring.profiles.active=server-1`)
```
---
title: Eureka配置客户端
toc: true
layout: springcloud
type: springcloud
categories:
  - SpringCloud
tags:
  - SpringCloud
  - Java
date: 2019-04-03 13:25:10
---
{{title}}
<!-- more -->
# `SpringCloud Greenwich` 版本集成 `Config Client`(注册到`Eureka Serever`)   

## 添加相关依赖

- 加依赖 
  
```xml
<!-- 注册到 Eureka Server 时需要添加的依赖 start -->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<!-- 注册到 Eureka Server 时需要添加的依赖 end -->

<!-- ConfigClient start -->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
<!-- ConfigClient end -->
``` 

**注:** 需要添加 `spring-boot-starter-web` 否则启动会失败

## 添加配置文件

- 配置 `application.yml`

```yaml
server:
  port: 3011
```

- 新建 `bootstrap.yml`

```yaml
spring:
  application:
    name: microservice-foo # 对应 config server 所获取的配置文件的 {application}
  cloud:
    config:
      uri: http://localhost:3001/   # profile 对应 config server 所获取的配置文件中的 {profile}
      profile: dev                  # profile 对应 config 所获取的配置文件中的 {profile}
      label: master                 # 指定Git 仓库的分支,对应 config server 所获取的配置文件的 {label}
#      discovery:
#        enabled: true               # 是否从配置中心读取文件
#        service-id: microservice-config-server    # 服务名

eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:1001/eureka/
```

> `yml`

  - `spring.application.name`: 对应的 `config server` 所获取的配置文件的 `{application}`
  - `spring.cloud.uri`: 指定`Config Server`的地址,默认是`http://localhost:8888`
  - `spring.cloud.config.profile`:`profile`对应`Config Server` 所获取配置文件的`{profile}`
  - `spring.cloud.config.label`:指定`Git`仓库的分支,对应`Config Server`所获取配置文件的`{label}`

[KEY POINT]:  

  - 注意属性配置文件,以上属性为`bootstrap.yml`文件内容而不是`application.yml`文件.如果配置在`application.yml`中,改部分配置就不能获取配置的正确值.例如:`applicaiton.yml`默认`spring.cloud.config.uri`的默认值是`http://localhost:8888`,而并非是`http://localhost:3011`
  
  - `SpringCloud` 有一个 "引导上下文" 的概念,这是主应用程序上下文(Application Context) 的父上下文。引导上下文负责从配置服务器加载配置属性,一级解密外部配置文件中的属性。和主应用程序加载`application.*`(yml或properties)中的属性不同,引导上下文加载`bootstrap.*`中的属性。配置在`bootstrap.*`中的属性优先级更高,因此默认情况下他们不能被而本地配置覆盖

## 从`API`接口验证是否能够正确获取自定义配置  

```java
@RestController
public class ConfigClientConfigController {

  // 获取 Config Server 配置文件中的 spring.cloud.config.profile属性值
	@Value("${spring.cloud.config.profile}")
	private String profile;

	@GetMapping("/profile")
	public String getProfile(){
		return this.profile;
	}
}
```

**注:** 如果需要注册到`Eureka`,需要在启动类`SpringcloudConfigClientApplication`上添加`@EnableEurekaClient`注解(注册到`Eureka Server`)

## 测试 

访问地址: <a href="http://127.0.0.1:3011/profile" target="_blank">`http://localhost:3011/profile`</a>

## 配套代码

* GitHub: https://github.com/ColorCoinK/springcloud-learning/tree/master/springcloud-config-client

## 参考博客

* <a href="http://blog.didispace.com/spring-cloud-starter-dalston-3/" target="_blank">程序猿DD —— 分布式配置中心</a>

* <a href="http://www.ityouknow.com/springcloud/2017/05/25/springcloud-config-eureka.html" target="_blank">纯洁的微笑 —— 配置中心服务化和高可用</a>
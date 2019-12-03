---
title: Eureka服务消费者Ribbon
toc: true
layout: springcloud
type: springcloud
categories:
  - SpringCloud
tags:
  - SpringCloud
  - Java
date: 2019-04-03 13:24:38
---
Eureka使用Ribbon消费RESTful API
<!-- more -->

# 基于Eureka搭建Ribbon服务消费者

> 添加`pom`

```xml
	<parent>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-netflix</artifactId>
		<version>1.4.6.RELEASE</version>
		<relativePath>..</relativePath> <!-- lookup parent from repository -->
	</parent>

	<properties>
		<main.basedir>${basedir}/../..</main.basedir>
	</properties>
	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-autoconfigure-processor</artifactId>
			<optional>true</optional>
		</dependency>
	</dependencies>
```

> 修改启动类

```java
@EnableDiscoveryClient
@SpringBootApplication
public class EurekaRibbonApplication {

	@Bean
	@LoadBalanced
	public RestTemplate restTemplate(){
		// Ribbon + RestTemplate 的重试
		SimpleClientHttpRequestFactory simpleClientHttpRequestFactory =new SimpleClientHttpRequestFactory();
		simpleClientHttpRequestFactory.setConnectTimeout(1000);
		simpleClientHttpRequestFactory.setReadTimeout(1000);
		return new RestTemplate(simpleClientHttpRequestFactory);
	}

	public static void main(String[] args) {
		SpringApplication.run(EurekaRibbonApplication.class, args);
	}

}
```

> `application.yml`配置文件

```yaml
server:
  port: 3001

spring:
  application:
    name: microservice-consumer-ribbon
  cloud:
    loadbalancer:
      retry:
        enabled: true
eureka:
  client:
    service-url:
      defaultZone: http://localhost:1001/eureka/
  instance:
    instance-id: ${spring.cloud.client.ipAddress}:${spring.application.name}:${server.port}
#    prefer-ip-address: true
ribbon:
  MaxAutoRetries: 1                 # 同一实例最大重试次数,不包括首次调用
  MaxAutoRetriesNextServer: 2       # 重试其他实例的最大重试次数,不包括首次所选的server
  OkToRetryOnAllOperations: false   # 是否所有操作都进行重试
```

* 需要向注册到Eureka服务中心

## 相关代码块

> Entity

```java
@Data
public class Book {

	private Long id;

	private String name;

	private Double price;

}

@Data
public class User {

	private Long id;

	private String name;

	private Integer age;

}
```

> Service

```java
@Service
public class RibbonService {

	@Autowired
	private RestTemplate restTemplate;


	public User queryUserInfoBy(Long id) {
		return this.restTemplate.getForObject("http://microsevice-provider-user/users/"+id, User.class);
	}

	public Book queryBookInfoById(Long id) {
		return this.restTemplate.getForObject("http://microsevice-provider-user/"+id, Book.class);
	}
}
```

> Controller

```java
@RestController
@RequestMapping("ribbon")
public class RibbonController {

	@Autowired
	private RibbonService ribbonService;


	@GetMapping("/user/{id}")
	public User queryUserInfoById(@PathVariable Long id) {
		return this.ribbonService.queryUserInfoBy(id);
	}

	@GetMapping("/book/{id}")
	public Book queryBookInfoById(@PathVariable Long id) {
		return this.ribbonService.queryBookInfoById(id);
	}
}
```

**注：**

1. 使用`RestTemplate` 调用 `API` 时,`url`为服务提供者的 `spring.application.name` 而不是 `ip:port`;

2. 调用的 `url` 需要确保无误,否则会调用失败.

- 实现了`Ribbon + RestTemplate` 重试

参考 <a href="http://www.itmuch.com/spring-cloud-sum/spring-cloud-retry/" target="_blank">周立-SpringCloud 各组件重试总结</a>

# 配套代码

* GitHub: https://github.com/ColorCoinK/springcloud-learning/tree/master/eureka-ribbon

# 参考文章

* <a href="https://blog.battcn.com/2017/07/26/springcloud/dalston/spring-cloud-ribbon/" target="_blank">唐亚峰 —— 服务消费者(Ribbon)</a>
* <a href="http://blog.didispace.com/spring-cloud-starter-dalston-2-2/" target="_blank">程序猿DD —— 服务消费者(Ribbon)[Dalston版]</a>
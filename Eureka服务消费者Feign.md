---
title: Eureka服务消费者Feign
toc: true
layout: springcloud
type: springcloud
categories:
  - SpringCloud
tags:
  - SpringCloud
  - Java
  - Eureka
date: 2019-04-03 13:24:26
---
Eureka使用Feign实现调用API
<!-- more -->

# 基于Eureka搭建Feign消费者

> 添加依赖

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
      <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>

    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-openfeign</artifactId>
    </dependency>

    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
    </dependency>

    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-devtools</artifactId>
      <scope>runtime</scope>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-autoconfigure</artifactId>
    </dependency>

    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <optional>true</optional>
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

> 启动类增加注解

```java
@EnableFeignClients
@EnableCircuitBreaker
@EnableDiscoveryClient
@SpringBootApplication
public class EurekaFeignApplication {

	@Bean
	@LoadBalanced
	public RestTemplate restTemplate(){
		return new RestTemplate();
	}

	public static void main(String[] args) {
		SpringApplication.run(EurekaFeignApplication.class, args);
	}

}
```

* `@EnableCircuitBreaker`: 服务降级
* `@EnableFeignClients`: Feign客户端
* `@EnableDiscoveryClient`: 注册到Eureka服务中心

- 上三个注解可以使用`@SpringCloud`注解代替

## 配置文件

```yaml
server:
  port: 4001

spring:
  application:
    name: microservice-consumer-feign
eureka:
  client:
    service-url:
      defaultZone: http://localhost:1001/eureka/
  instance:
    instance-id: ${spring.cloud.client.ipAddress}:${spring.application.name}:${server.port}
#    prefer-ip-address: true

feign:
  hystrix:
    enabled: true   # 全局启用 Hystrix,默认禁用
management:
  endpoint:
    web:
      exposure:
        include: 'hystrix.stream'
    health:
      show-details: always
  ribbon:
    eager-load:
      enabled: true
      clients: microsevice-provider-user
    MaxAutoRetries: 1                 # 同一实例最大重试次数,不包括首次调用
    MaxAutoRetriesNextServer: 2       # 重试其他实例的最大重试次数,不包括首次所选的server
    OkToRetryOnAllOperations: false   # 是否所有操作都进行重试
```

## 代码块

### 实体类

> `Book`

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Book {

	private Long id;

	private String name;

	private Double price;

}
```

> User

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {

	private Long id;

	private String name;

	private Integer age;

}
```

### Client

> `BookFeignClient`

```java
/**
 *@ClassName BookFeignClient
 *@Description <br/>
 *  获取造成fallback的原因, 同时实现服务降级
 *@Author Dew
 *@Date 2019/2/15 9:50
 *@Version 1.0
 **/
@FeignClient(name = "microsevice-provider-user",fallbackFactory = BookFeignClientFallbackFactory.class)
public interface BookFeignClient {


	@GetMapping("/{id}")
	Book queryBookInfoById(@PathVariable("id") Long id);


	/**
	 * @Title
	 * @Description 获得造成fallback的原因
	 * @return
	 **/
	@Component
	@Slf4j
	class BookFeignClientFallbackFactory implements FallbackFactory<BookFeignClient> {

		@Override
		public BookFeignClient create(Throwable cause) {
			return new BookFeignClient() {
				@Override
				public Book queryBookInfoById(Long id) {
					log.error("进入回退逻辑", cause);
					return new Book(id, "领域驱动设计", 89.99);
				}
			};
		}
	}
}
```

> `UserFeignClient`

```java
/**
 * @ClassName UserFeignClient
 * @Description <br/> 
 * 服务提供者调用类 \n `@FeignClient`的 name、url 等属性支持占位符,例如(`@FeignClient(name
 * @Author Dew
 * @Date 2019/2/14 16:33
 * @Version 1.0
 **/
@FeignClient(name = "microsevice-provider-user", fallback = UserFeignClientFallback.class)
public interface UserFeignClient {


	@GetMapping("/users/{id}")
	User queryUserInfoById(@PathVariable("id") Long id);


	/**
	 * @return com.lerning.entity.User
	 * @Title queryUserInfoByIdAndName
	 * @Description GET 请求多参数的URL写法一
	 * <link src="http://www.itmuch.com/spring-cloud-sum/feign-multiple-params/" />
	 * @Param
	 **/
	@GetMapping("/users/findByIdAndName")
	User queryUserInfoByIdAndName(@RequestParam("id") Long id,
			@RequestParam("username") String userName);

	@GetMapping("/users/findByIdAndName")
	User queryUserInfoByIdAndName(@RequestParam Map<String, Object> param);

	// 服务降级
	@Component
	class UserFeignClientFallback implements UserFeignClient {

		@Override
		public User queryUserInfoById(Long id) {
			return new User(id, "默认用户", 1);
		}


		@Override
		public User queryUserInfoByIdAndName(Long id, String userName) {
			return new User(id, "默认用户", 1);
		}

		@Override
		public User queryUserInfoByIdAndName(Map<String, Object> param) {
			return new User(1L, "默认用户", 1);
		}
	}
}
```

### Controller

```java
/**
 *@ClassName 
 *@Description <br/>
 *  测试Feign调用Controller
 *@Author Dew
 *@Date  2019/2/14 14:42
 *@Version 1.0
 **/
@RestController
@RequestMapping("/feign")
public class UserController {

	@Autowired
	private UserFeignClient userFeignClient;

	@Autowired
	private BookFeignClient bookFeignClient;

	@GetMapping("/user/{id}")
	public User findById(@PathVariable Long id) {
		return this.userFeignClient.queryUserInfoById(id);
	}

	@GetMapping("/user/findByIdAndName")
	public User queryUserInfoByIdAndName(@RequestParam("id") Long id,
			@RequestParam("username") String userName) {
		// GET 请求多参数的 URL,参数传递方式一
		// return this.userFeignClient.queryUserInfoByIdAndName(id, userName);

		// GET 请求多参数的 URL 参数传递方式二
		Map<String, Object> param = new HashMap<String, Object>();
		param.put("id", id);
		param.put("username", userName);
		return this.userFeignClient.queryUserInfoByIdAndName(param);
	}

	@GetMapping("/book/{id}")
	public Book queryBookInfoById(@PathVariable Long id) {
		return this.bookFeignClient.queryBookInfoById(id);
	}

}
```

## 自定义`Feign`自定义配置[细粒度配置]

> 自定义日志级别 

- 默认 Feign 是不打印任何日志的,开启 Feign 日志,Feign 日志有四种级别
  
  * NONE[性能最佳,适用于生产]: 不记录任何日志(默认值)
  * BASIC[适用于生产环境追踪问题]: 仅记录请求方法、URL、响应状态码及响应时间
  * HEADERS: 记录 BASIC 级别的基础上,记录请求和响应的 header
  * FULL[比较适用于开发及测试环境定位问题]: 记录请求和响应的 header、body 和元数据

* 配置参考博客 <a href="http://www.itmuch.com/spring-cloud/finchley-10/" target="_blank">Feign配置自定义`[细粒度配置]`</a>

> `Feign` 使用 `Hystrix` 配置

- 配置 Feign 相关 Hystrix 配置


* 配置参考博客 <a href="http://www.itmuch.com/spring-cloud/finchley-14/" target="_blank">周立-`Feign使用Hystrix`</a>

<!-- > 配置监控端点并实现可视化监控数据   -->

> `Ribbon` 的饥饿加载(eager-load)模式  

Problems:  

  在使用`SpringCloud`的`Ribbon`或`Feign`来实现服务调用的时候,如果我们的机器或网络环境等原因不是很好,有时候后会出现
  : 服务消费方调用服务提供方接口时,第一次请求经常会超时,而之后调用就没有问题了.
  
 
 * 解决方式参考 <a href="http://blog.didispace.com/spring-cloud-tips-ribbon-eager/" target="_blank">翟永超-Ribbon的饥饿加载模式</a> 
 
 
 > `Feign` 的重试
 
 参考 <a href="http://www.itmuch.com/spring-cloud-sum/spring-cloud-retry/" target="_blank">周立-SpringCloud 各组件重试总结</a>

## 配套代码

* GitHub: https://github.com/ColorCoinK/springcloud-learning/tree/master/eureka-feign

> 参考博客

* <a href="http://www.itmuch.com/spring-cloud/finchley-9/">周立 —— Feign</a>

> 推荐阅读

* <a href="http://blog.didispace.com/springcloud2/" target="_blank">程序员DD(翟永超) —— 服务消费者</a>

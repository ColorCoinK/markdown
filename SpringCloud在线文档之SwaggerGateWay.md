---
title: SpringCloud在线文档之SwaggerGateWay
toc: true
layout: springcloud
type: springcloud
categories:
  - SpringCloud
tags:
  - SpringCloud
  - Swagger
  - GateWay
  - Java
date: 2019-04-03 13:26:53
---
使用SpringCloud构建分布式在线文档
<!-- more -->

# Swagger-service-a

> 依赖

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.14.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
  </parent>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <java.version>1.8</java.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-eureka</artifactId>
    </dependency>
    <!-- Swagger start -->
    <dependency>
      <groupId>com.spring4all</groupId>
      <artifactId>swagger-spring-boot-starter</artifactId>
      <version>1.7.0.RELEASE</version>
    </dependency>
    <!-- Swagger end -->
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
        <version>Dalston.SR1</version>
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
      <plugin>
        <artifactId>maven-compiler-plugin</artifactId>
        <configuration>
          <source>${java.version}</source>
          <target>${java.version}</target>
        </configuration>
      </plugin>
    </plugins>
  </build>
```

> 配置文件

```yaml
server:
  port: 10010
spring:
  application:
    name: swagger-service-a
eureka:
  client:
    service-url:
      defaultZone: http://localhost:1001/eureka/
swagger:
  base-package: com.learning
```

> 启动类

```java
@EnableSwagger2Doc
@EnableDiscoveryClient
@SpringBootApplication
public class SwaggerServiceAApplication {

	public static void main(String[] args) {
		SpringApplication.run(SwaggerServiceAApplication.class, args);
	}
}
```

> Controller

```java
@RestController
public class ApiController {

	@Autowired
	DiscoveryClient discoveryClient;

	@GetMapping("service-a")
	public String dc() {
		String services = "Services:" + discoveryClient.getServices();
		System.out.println(services);
		return services;
	}
}
```

- 代码地址: https://github.com/ColorCoinK/springcloud-learning/tree/master/swagger-service-a

# Swagger-service-b 

* 在简化的情况下,代码与`Swagger-service-a`一致,通过`@GetMappning`的标识区分两个不同服务

- 代码地址: https://github.com/ColorCoinK/springcloud-learning/tree/master/swagger-service-b

# 网关整合 SpringCloud(Dalston.SR5 版本) 集成 Swagger API 文档配置类

- 需要启动 Eureka-Server 服务注册中心
- 采用`Spring4all社区版的自定义Swagger包`,其他版本依赖可能与当前使用方法不一致,请查找其他资料

## 项目创建

 1. 创建`SpringBoot`项目,本示例选择`spring-boot-starter-parent 1.5.12.RELEASE`、`spring-cloud-dependencies Dalston.SR5`;
 2. 使用`Eureka`做服务注册中心
 3. 使用`zuul` 路由配置

## 导入依赖

```xml
  <dependency>
    <groupId>com.spring4all</groupId>
    <artifactId>swagger-spring-boot-starter</artifactId>
    <version>1.7.0.RELEASE</version>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
  </dependency>
```

## 启动类增加注解

```java
@EnableSwagger2Doc
@EnableZuulProxy
@SpringCloudApplication
public class ApiGatewayApplication {

	public static void main(String[] args) {
		SpringApplication.run(ApiGatewayApplication.class, args);
	}
}
```

## 创建配置文件类,配置`API`文档

```java
/**
 * API 网关整合 Swagger API 文档配置类
 *
 * @ClassName: DocumentationConfig
 * @Description: TODO
 * @Created by Dew on 2018/07/18
 */
@Component
@Primary
public class DocumentationConfig implements SwaggerResourcesProvider {

	@SuppressWarnings({"unchecked", "rawtypes"})
	@Override
	public List<SwaggerResource> get() {
		List resources = new ArrayList<>();
		resources.add(swaggerResource("service-a", "/swagger-service-a/v2/api-docs", "2.0"));
		resources.add(swaggerResource("service-b", "/swagger-service-b/v2/api-docs", "2.0"));
		return resources;
	}

	private SwaggerResource swaggerResource(String name, String location, String version) {
		SwaggerResource swaggerResource = new SwaggerResource();
		swaggerResource.setName(name);
		swaggerResource.setLocation(location);
		swaggerResource.setSwaggerVersion(version);
		return swaggerResource;
	}
}
```

## 配置文件,设置注册中心,项目端口等

```yaml
spring:
  application:
    name: api-gateway

server:
  port: 1101

eureka:
  client:
    service-url:
      defaultZone: http://localhost:1001/eureka/
```

## 参考文章

* <a href="http://blog.didispace.com/Spring-Cloud-Zuul-use-Swagger-API-doc/">程序猿DD —— Spring Cloud Zuul中使用Swagger汇总API接口文档</a>
* 
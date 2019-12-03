---
title: Eureka服务提供者
toc: true
layout: springcloud
type: springcloud
categories:
  - SpringCloud
tags:
  - SpringCloud
  - Java
  - Eureka
date: 2019-04-03 13:05:05
---
搭建Eureka服务提供者,将服务注册到服务中心
<!-- more -->

# 基于 Eureka 搭建服务提供者

本项目是一个基于 `SpringBoot 1.5.12.RELEASE`、`SpringCloud Dalston.SR5`、`H2`、`Eureka` 的服务提供者.

> 添加`pom`依赖

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.12.RELEASE</version>
    <relativePath/> 
  </parent>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <java.version>1.8</java.version>
    <spring-cloud.version>Edgware.SR5</spring-cloud.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter</artifactId>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-devtools</artifactId>
      <scope>runtime</scope>
    </dependency>

    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <optional>true</optional>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <dependency>
      <groupId>com.h2database</groupId>
      <artifactId>h2</artifactId>
      <scope>runtime</scope>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-autoconfigure</artifactId>
    </dependency>

    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-eureka</artifactId>
    </dependency>

    <!-- 配置 SpringBoot 监控功能 -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-actuator</artifactId>
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

> `@EnableDiscoveryClient`启动类

```java
@EnableDiscoveryClient
@SpringBootApplication
public class EurekaProviderApplication {

	public static void main(String[] args) {
		SpringApplication.run(EurekaProviderApplication.class, args);
	}

}
```

> 初始化数据

- 使用`H2`内存数据库初始化测试数据

**注:**
 
1. `SpringBoot 2.x`不能像 1.x 一样,使用 `spring.datasource.schema/data` 指定初始化脚本.否则与 `actuator` 不能共存
2. 项目中的依赖并不是与[参考的博客一样,自行选择]
3. 项目参考来源`http://book.itmuch.com/2 Spring Cloud/2.2 服务提供者.html`

* 建表语句(schema.sql)

```sql
drop table if exists tbl_book;

create table tbl_book(
id number(12) primary key not null,
name varchar not null,
price decimal not null
);

create table t_user(
id number(12) primary key not null,
name varchar not null,
age int not null
);

-- DELETE FROM tbl_book;
```

* 插入初始化数据(data.sql)

```sql
INSERT INTO tbl_book (id, name, price) VALUES (1, 'Spring Boot - Spring Data JPA with Hibernate and H2 Web Console', 0.0);
INSERT INTO tbl_book (id, name, price) VALUES (2, 'Spring Boot - Spring Data JPA with Hibernate and H2 Web Console 2', 1.0);
INSERT INTO tbl_book (id, name, price) VALUES (3, 'Spring Boot - Spring Data JPA with Hibernate and H2 Web Console 3', 2.0);
INSERT INTO tbl_book (id, name, price) VALUES (4, 'Spring Boot - Spring Data JPA with Hibernate and H2 Web Console 4', 3.0);

insert into t_user(id, name, age) values(1,'JSON',29);
insert into t_user(id, name, age) values(2,'Alibaba',25);
insert into t_user(id, name, age) values(3,'Apache',30);
insert into t_user(id, name, age) values(4,'GitHub',20);
insert into t_user(id, name, age) values(5,'Sun',18);
```

## 配置文件

```yml
server:
  port: 2001

spring:
  application:
    name: microsevice-provider-user
  datasource:                           # 指定数据源;默认H2建表脚本(根目录/schema.sql);默认H2的insert脚本(classpath:data.sql)
    url: jdbc:h2:mem:~/example-app;     # 指定数据库,默认
    platform: h2
    username: sa
    password:
    driver-class-name: org.h2.Driver
  h2:
    console:
      enabled: false # 是否启用H2数据库控制台
      settings:
        web-allow-others: false         # 允许远程浏览器访问H2 数据库控制台
        trace: false
  jpa:
    show-sql: true
    hibernate:
      ddl-auto: none
    generate-ddl: false
    database-platform: org.hibernate.dialect.H2Dialect

logging:
  level:
    root: INFO
    org.hibernate: INFO
    org.hibernate.type.descriptor.sql.BasicBinder: TRACE
    org.hibernate.type.descriptor.sql.BasicExtractor: TRACE

eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:1001/eureka/
  instance:
    instance-id: ${spring.cloud.client.ipAddress}:${spring.application.name}:${server.port} # 自定义微服务的 `instance-id`,当前配置的默认值${spring.cloud.client.ipAddress}:${spring.application.name}:${server.port}
#    prefer-ip-address: true
#    lease-expiration-duration-in-seconds: 30
#    lease-renewal-interval-in-seconds: 10
```

## Entity

```java
@Entity
@Data
@Table(name = "tbl_book")
public class Book {

	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Long id;

	@Column
	private String name;

	@Column
	private Double price;

}
```

## Repository

```java
@Repository
public interface BookRepository extends JpaRepository<Book,Long> {

}
```

## Controller

```java
@RestController
public class BookController {

	@Autowired
	private BookRepository bookRepository;

	@GetMapping("/{id}")
	public Book findById(@PathVariable Long id) {
		return this.bookRepository.findOne(id);
	}

}
```

## `Eureka` 常见问题配置参考链接  

<a href="http://www.itmuch.com/spring-cloud-sum-eureka/" target="_blank">Spring Cloud中，Eureka常见问题总结(周立)</a>

## 实现监控(SpringBoot Actuator) 

- 添加`Actutor` 监控(SpringBoot 1.x 版本)

```xml
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
```

- 参考示例: 
  
<a href="http://www.itmuch.com/spring-cloud/finchley-3/" target="_blank">周立 —— SpringBoot 集成Actutor</a>

**注：** 

1. `SpringBoot 1.x`版本访问`http://10.18.13.2:2001/health`,`SpringBoot 2.x`访问`http://10.18.13.2:2001/actutor/health`

2. 通过 `spring.appliccation.name:server.port` 访问 API 时,需要在 `C:\Windows\System32\drivers\etc\hosts` 文件夹中添加

```
127.0.0.1 microsevice-provider-user
```
---
title: SpringCloud鉴权之Token
toc: true
layout: springcloud
type: springcloud
categories:
  - SpringCloud
tags:
  - SpringCloud
  - Token
  - Java
date: 2019-04-03 13:25:45
---
{{title}}
<!-- more -->

# 基于 Eureka 搭建鉴权服务(jjwt Token认证)

> 前提

  * 需要启动服务注册中心,eureka-server,项目地址:<a href="https://github.com/ColorCoinK/springcloud-learning/tree/master/eureka-server" target="_blank"></a>

## 说明
  
  * 只是码了这部分代码,原理及详细的解释可阅读尾部的链接地址

## 项目结构

```
src-main
      |
      |
    java
      |
       ---  entity
              |
               -- dto
                   |
                    -- UserDto  Token 刷新时间实体类
              |
               -- UserPO  用户数据库映射实体类
              |
              |
            enumerate
              |
               -- ParseToken Token 包含的内容解析
              |
            repository
              |
                -- UserRepository 用户数据库相关操作
              |
            utils
              |
               -- JwtUtil 生成、解析、刷新 Token 工具类
              |
            EurekaTokenServerApplication
      |
    resources
      |
        -- application.yml  主配置文件
        -- data.sql 初始化数据SQL
        -- schema.sql 建表语句SQL
      

```

## 功能概述

1. Eureka 服务注册中心
2. 签发Token

## 结构分析

1. Eureka 服务注册中心
2. Eureka 客户端
3. jjwt 签发token
4. H2 内存数据库
5. JPA 操作数据库

## 配置相关

> 依赖配置

```xml
 <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.19.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
  </parent>
  <groupId>com.learning</groupId>
  <artifactId>eureka-token-server</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <name>eureka-token-server</name>
  <description>Eureka Token server</description>

  <properties>
    <java.version>1.8</java.version>
    <spring-cloud.version>Edgware.SR5</spring-cloud.version>
  </properties>

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

  <dependencies>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>

    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>

    <!-- token starts    -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <dependency>
      <groupId>io.jsonwebtoken</groupId>
      <artifactId>jjwt</artifactId>
      <version>0.9.0</version>
    </dependency>

    <!-- token end -->

    <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>fastjson</artifactId>
      <version>1.2.56</version>
    </dependency>

    <dependency>
      <groupId>com.h2database</groupId>
      <artifactId>h2</artifactId>
      <scope>runtime</scope>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
      <groupId>com.h2database</groupId>
      <artifactId>h2</artifactId>
      <scope>runtime</scope>
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
```

> 启动类

* 向`Eureka`服务中心注册,需要使用`@EnableDiscoveryClient`标记为客户端

```java
@EnableDiscoveryClient
@SpringBootApplication
public class EurekaTokenServerApplication {

	public static void main(String[] args) {
		SpringApplication.run(EurekaTokenServerApplication.class, args);
	}

}
```

> 配置文件

```yaml
server:
  port: 1011
spring:
  application:
    name: token-server
  datasource:
    url: jdbc:h2:mem:~/example-app
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
  instance:
    instance-id: ${spring.cloud.client.ipAddress}:${spring.application.name}:${server.port}
  client:
    service-url:
      defaultZone: http://localhost:1001/eureka/

```

> 初始化数据

>> `data.sql`

```sql
insert into tbl_account (id,name,age,sex,password,role) values (1001,'admin',18,'男','1','ROLE_ADMIN');
insert into tbl_account (id,name,age,sex,password,role) values (1002,'zy',25,'男','1','ROLE_USER');
insert into tbl_account (id,name,age,sex,password,role) values (1003,'lb',18,'男','1','ROLE_USER');
insert into tbl_account (id,name,age,sex,password,role) values (1004,'lk',22,'男','1','ROLE_USER');
insert into tbl_account (id,name,age,sex,password,role) values (1005,'zj',18,'男','1','ROLE_USER');
insert into tbl_account (id,name,age,sex,password,role) values (1006,'lc',21,'男','1','ROLE_USER');
insert into tbl_account (id,name,age,sex,password,role) values (1007,'wdd',18,'男','1','ROLE_USER');
insert into tbl_account (id,name,age,sex,password,role) values (1008,'ln',18,'男','1','ROLE_USER');
insert into tbl_account (id,name,age,sex,password,role) values (1009,'gy',20,'女','1','ROLE_USER');
insert into tbl_account (id,name,age,sex,password,role) values (1010,'dew',35,'男','1','ROLE_USER');
```

>> `schema.sql`

```sql
drop table if exists tbl_account;

create table tbl_account(
  id number(12) primary  key  not null ,
  name varchar not null,
  age int(2) not null ,
  sex char(2) not null ,
  password varchar(80) not null ,
  role varchar(10) not null
);
```

## 代码块

> 数据映射实体类

```java
package com.learning.entity;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.Table;
import lombok.Data;
import lombok.ToString;

/**
 * @Title 用户 —— 数据库映射实体类
 * @ClassName Account
 * @Description tbl_account
 * @Author sanss
 * @Date 2019/2/22 13:57
 * @Version 1.0
 */
@Entity
@Data
@ToString
@Table(name = "tbl_account")
public class UserPO {


	@Id
	@GeneratedValue
	private Long id;

	/**
	 * 姓名
	 **/
	private String name;

	/**
	 * 年龄
	 **/
	private Integer age;

	/**
	 * 性别
	 **/
	private String sex;

	/**
	 * 登录密码
	 **/
	private String password;

	/**
	 * 角色名称
	 **/
	private String role;

}
```

> 数据传输实体类

```java
/**
 * @Title token 刷新时间数据传输实体类
 * @ClassName UserDo
 * @Description TODO
 * @Author sanss
 * @Date 2019/2/22 16:24
 * @Version 1.0
 */
@Data
public class UserDto extends UserPO {

	/**
	 * 有效时长(unix 时间戳)
	 **/
	private Long expritaion;

	private String token;

	public UserDto() {
		this.expritaion = (long) 1000 * 60 * 10;
	}

	public UserDto(String token, Long id, String name) {
		this.token = token;
		this.setId(id);
		this.setName(name);
	}
}
```

> 枚举

```java
/**
 * 解析token,返回什么值
 *
 * @author sanss
 * @create 2019/2/22
 * @since 1.0.0
 */
public enum ParseToken {
	/**
	 * 用户编号
	 **/
	UID,
	/**
	 * 登录名
	 **/
	USER_NAME,
	/**
	 * 实体类
	 **/
	USER_DTO;

}
```

> `Repository`

```java
/**
 * @Title 用户数据库操作类
 * @ClassName UserRepository
 * @Description TODO
 * @Author sanss
 * @Date 2019/2/22 15:11
 * @Version 1.0
 */
@Repository
public interface UserRepository extends JpaRepository<UserPO, Long> {

	/**
	 * @Title findByNameAndPassword
	 * @Description   登录验证
	 * @Param
	     * @param name  登录名
		 * @param password  密码
	 * @return com.learning.entity.Account
	 **/
	UserPO findByNameAndPassword(String name, String password);

}
```

## `Token`工具类

```java
import com.learning.entity.dto.UserDto;
import com.learning.enumerate.ParseToken;
import io.jsonwebtoken.Claims;
import io.jsonwebtoken.JwtBuilder;
import io.jsonwebtoken.JwtException;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;

/**
 *@ClassName JwtUtil
 *@Description <br/>
 *  JSON Web Token 生成、解析、刷新工具类
 *@Author Dew
 *@Date
 *@Version 1.0
 **/
public class JwtUtil {

	/**
	 * token 加密算法 HS256
	 **/
	private final static SignatureAlgorithm ENCRYPT_KEY = SignatureAlgorithm.HS256;

	/**
	 * 该JWT 所面向的用户
	 **/
	private final static String SUBJECT = "dew";

	/**
	 * 私钥
	 **/
	private final static String SECRET_KEY = "secretkey";

	/**
	 * @param uid 用户id
	 * @param name 用户名
	 * @param duration 有效时长(unix时间戳)
	 * @return java.lang.String
	 * @Title createJws
	 * @Description
	 * @Param
	 **/
	public static String createJws(Long uid, String name, String password, long duration) {
		long nowMillis = System.currentTimeMillis();
		Date now = new Date(nowMillis);

		// 自定义token携带的参数值
		Map<String, Object> claims = new HashMap<>();
		claims.put("uid", uid);
		claims.put("user_name", name);
//		claims.put("password", password);

		JwtBuilder builder = Jwts.builder()
				                     // 该JWT 所面向的用户
				                     .setSubject(SUBJECT)
				                     // token 携带的参数
				                     .setClaims(claims)
				                     // token创建时间,unix时间戳格式
				                     .setIssuedAt(now)
				                     .signWith(ENCRYPT_KEY, SECRET_KEY);
		if (duration >= 0) {
			// 设置最大有效时间,unix时间戳
			long expirationMills = nowMillis + duration;
			Date expirationTime = new Date(expirationMills);
			//expire 指定 token 的声明周期,unix 时间戳格式
			builder.setExpiration(expirationTime);
		}

		return builder.compact();
	}

	/**
	 * @return java.lang.Object
	 * @Title analysisToken
	 * @Description 解析 Token 内容信息
	 * @Param
	 **/
	public static Object analysisToken(String token, ParseToken type) {

		try {
			// 解析 token 中携带的信息
			Claims claims = parseJWs(token);

			long now = System.currentTimeMillis();
			// token 失效时间(设置的过期时间为 java.util.Date)
			Date expDate = claims.get("exp", Date.class);
			long exp = expDate.getTime();
			// 用户编号
			Long uid = claims.get("uid", Long.class);
			// 登录名
			String userName = claims.get("user_name", String.class);

			// 1.token 过期
			if (exp < now) {
				throw new JwtException("Token 过期");
			}
			// 2.在有效期内,刷新 token
			switch (type) {
				case UID:
					return uid;
				case USER_DTO:
					UserDto userDto = new UserDto(token, uid, userName);
					exp = now + new UserDto().getExpritaion();
					userDto.setExpritaion(exp);
					return userDto;
				case USER_NAME:
					return userName;
				default:
					break;
			}

		} catch (JwtException e) {
			throw new JwtException(e.getMessage());
		} catch (Exception exception) {
			throw new RuntimeException("token解析错误");
		}
		return false;
	}

	/**
	 * @return io.jsonwebtoken.Claims
	 * @Title parseJWs
	 * @Description 解析 token 字符串
	 * @Param
	 **/
	static Claims parseJWs(String token) throws JwtException {
		// 解析 token 中携带的信息
		Claims claims = Jwts.parser()
				                .setSigningKey(SECRET_KEY)
				                .parseClaimsJws(token)
				                .getBody();

		return claims;
	}

}
```

## 测试类

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class JwtUtilTest {


	@Test
	public void jwsTest() {
		//1001,'admin',18,'男','1','ROLE_ADMIN'
		System.out.println(JwtUtil.createJws(1001L, "admin", "admin", 1000 * 60 * 10));
	}

	@Test
	public void analysisJwsTest() {
		Claims claims = new JwtUtil().parseJWs(
				"eyJhbGciOiJIUzI1NiJ9.eyJ1aWQiOjEwMDEsInBhc3N3b3JkIjoiYWRtaW4iLCJ1c2VyX25hbWUiOiJhZG1pbiIsImV4cCI6MTU1MDgyMDcwOSwiaWF0IjoxNTUwODIwMTA5fQ.sh0ppz6cDOlK0ewlC0-cCFkyOV_FsVfvCTR4JzrmRTI");
		System.out.println(claims);
	}
}
```

## 配套代码

> 代码地址

* GitHub: https://github.com/ColorCoinK/springcloud-learning/tree/master/eureka-token-server

> 参考文章列表

1. <a href="https://blog.battcn.com/2017/08/15/springcloud/dalston/spring-cloud-security-jwt/" target="_blank"> 唐亚峰 —— SpringCloud-服务认证(JWT)</a>
2. <a href="https://www.javazhiyin.com/35020.html" target="_blank">Java知音 - 单点登录原理与简单实现</a>
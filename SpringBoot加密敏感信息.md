---
title: SpringBoot加密敏感信息
toc: true
layout: springboot
type: "springboot"
categories:
  - SpringBoot
tags:
  - SpringBoot
  - Java
date: 2019-11-08 13:26:17
---
在SpringBoot中,使用`jasypt`依赖实现配置文件加密。记录了`Hikari`与`Druid`连接池的使用方法,配置文件中使用`ENC(xx)`方式,`extends HikariDataSource`的两种方式实现
<!-- more -->

# <h3>SpringBoot 配置加密的数据库连接信息</h3>

## 使用`Hikari`数据库连接池

* `application.yml` 配置文件(根据自身实际情况调整配置内容及配置文件格式`properties`)

```yml
spring:
  datasource:
    primary:
      driver-class-name: com.cloudera.impala.jdbc41.Driver
      jdbc-url: xx
    secondary:
      driver-class-name: oracle.jdbc.OracleDriver
      jdbc-url: xx
      username: xx
      password: xx

# 配置自己的根密码
jasypt:
  encryptor:
    password: xx 
```

* 两种方式都需要导入`jasypt`依赖,导入配置后需要在配置文件中添加`jasypt.encryptor.password`属性

1. 添加依赖

  ```xml
  <dependency>
    <groupId>com.github.ulisesbocchio</groupId>
    <artifactId>jasypt-spring-boot-starter</artifactId>
    <version>2.1.2</version>
  </dependency>
  ```

2. 使用`jasypt`包中的依赖,生成加密后的字符串

  * 这里是在测试类中生成的

  ```java

  @Autowired
  private StringEncryptor encryptor;

  @Test
	public void generatorEncryptProperties() {
		/* 生成加密的数据库配置信息 */
		// dev
		String jdbcUrlDev = "jdbc:oracle:thin:@//127.0.0.1:1521/db";
		jdbcUrlDev = encryptor.encrypt(jdbcUrlDev);

		String userNameDev = "test";
		userNameDev = encryptor.encrypt(userNameDev);

		String passwordDev = "test";
		passwordDev = encryptor.encrypt(passwordDev);

		log.info("jdbcUrlDev:{}", jdbcUrlDev);
		log.info("userNameDev:{}", userNameDev);
		log.info("passwordDev:{}", passwordDev);

		// prod
		String jdbcUrlPro = "jdbc:oracle:thin:@//ip:1521/db";
		jdbcUrlPro = encryptor.encrypt(jdbcUrlPro);

		String userNamePro = "test";
		userNamePro = encryptor.encrypt(userNamePro);

		String passwordPro = "test";
		passwordPro = encryptor.encrypt(passwordPro);

		log.info("jdbcUrlPro:{}", jdbcUrlPro);
		log.info("userNamePro:{}", userNamePro);
		log.info("passwordPro:{}", passwordPro);
	}
  ```

> 第一种(以上步骤是相同的,以下步骤开始区分)

3. 修改配置文件

* 配置文件中使用密文替换原先配置`ENC(xx)`

```yml
spring:
  datasource:
    primary:
      driver-class-name: com.cloudera.impala.jdbc41.Driver
      jdbc-url: ENC(xx)
    secondary:
      driver-class-name: oracle.jdbc.OracleDriver
      jdbc-url: ENC(xx)
      username: ENC(xx)
      password: ENC(xx)
```

4. 这是上述配置中`DataSource`配置

```java
import javax.sql.DataSource;

import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.jdbc.core.JdbcTemplate;

import com.zaxxer.hikari.HikariDataSource;
import org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate;

@Configuration
public class HikariDataSourceConfig {

	@Primary
	@Bean(name = "primaryDataSource")
	@Qualifier(value = "primaryDataSource")
	@ConfigurationProperties(prefix = "spring.datasource.primary")
	public DataSource primaryDataSource() {
		return DataSourceBuilder.create().type(HikariDataSource.class).build();
	}


	@Bean(name = "primaryJdbcTemplate")
	public JdbcTemplate primaryJdbcTemplate(@Qualifier("primaryDataSource") DataSource dataSource) {
		return new JdbcTemplate(dataSource);
	}

	@Bean(name = "secondaryDataSource")
	@Qualifier(value = "secondaryDataSource")
	@ConfigurationProperties(prefix = "spring.datasource.secondary")
	public DataSource secondaryDataSource() {
		return DataSourceBuilder.create().type(HikariDataSource.class).build();
	}

	@Bean(name = "secondaryJdbcTemplate")
	public NamedParameterJdbcTemplate secondaryJdbcTemplate(@Qualifier("secondaryDataSource") DataSource dataSource) {
		return new NamedParameterJdbcTemplate(dataSource);
	}
}

```

5. 创建测试类,测试程序是否能够获取数据库连接

```java
import java.util.HashMap;
import java.util.List;
import java.util.Map;

import lombok.extern.slf4j.Slf4j;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate;
import org.springframework.test.context.junit4.SpringRunner;

import com.alibaba.fastjson.JSON;

@Slf4j
@SpringBootTest
@RunWith(SpringRunner.class)
public class HikariDataSourceConfigTest {

	@Autowired
	private NamedParameterJdbcTemplate secondaryJdbcTemplate;

  @Test
	public void queryAdGeocodeTest() {
		log.info("secondary data source connection start:");
		String sql = "select 1 from dual";

		List<Map<String, Object>> result = secondaryJdbcTemplate.queryForList(sql,new HashMap<>());
		log.info("secondary data source:\t{}",JSON.toJSONString(result));
	}
	
}
```

* 控制台输出,以下是正常时的内容

```java
secondary data source connection start:
13:55:14.949 [main] INFO  com.zaxxer.hikari.HikariDataSource - HikariPool-1 - Starting...
13:55:15.209 [main] INFO  com.zaxxer.hikari.pool.PoolBase - HikariPool-1 - Driver does not support get/set network timeout for connections. (oracle.jdbc.driver.T4CConnection.getNetworkTimeout()I)
13:55:15.219 [main] INFO  com.zaxxer.hikari.HikariDataSource - HikariPool-1 - Start completed.
secondary data source:	[{"1":1}]
```

> 第二种

3. 新建`JasyptHikariDataSource`

- 注释的属性可以根据实际情况进行调整,选择自己需要加密的属性重写相关方法

```java
import com.zaxxer.hikari.HikariDataSource;
import lombok.extern.slf4j.Slf4j;
import org.jasypt.encryption.StringEncryptor;
import org.springframework.beans.factory.annotation.Autowired;

/**
 * @ClassName JasyptHikariDataSource
 * @Description <br/> 加密数据库密码
 * @Author Dew
 * @Date 2019/11/8 9:02
 * @Version 1.0
 **/
@Slf4j
public class JasyptHikariDataSource extends HikariDataSource {

	@Autowired
	private StringEncryptor encryptor;


	// @Override
	// public String getJdbcUrl() {
	// 	String encJdbcUrl = super.getJdbcUrl();
	// 	if (encJdbcUrl.isEmpty()) {
	// 		return null;
	// 	}

	// 	String decJdbcUUrl = encryptor.decrypt(encJdbcUrl);
	// 	log.info("解密后的jdbcUrl:\t{}", decJdbcUUrl);
	// 	return decJdbcUUrl;
	// }

	// @Override
	// public String getUsername() {
	// 	String encUsername = super.getUsername();
	// 	if (encUsername.isEmpty()) {
	// 		return null;
	// 	}

	// 	String decUsername = encryptor.decrypt(encUsername);
	// 	log.info("解密后的用户名:\t{}", decUsername);
	// 	return decUsername;
	// }

	@Override
	public String getPassword() {

		// 配置文件中加密后的密码
		String encPassword = super.getPassword();
		if (encPassword.isEmpty()) {
			return null;
		}

		String decPassword = encryptor.decrypt(encPassword);
		log.info("解密后的密码:\t{}", decPassword);
		return decPassword;
	}

}
```

4. 修改`HikariDataSource`中的数据源配置

```java
import javax.sql.DataSource;

import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.jdbc.core.JdbcTemplate;

import com.zaxxer.hikari.HikariDataSource;
import org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate;

@Configuration
public class HikariDataSourceConfig {

	@Primary
	@Bean(name = "primaryDataSource")
	@Qualifier(value = "primaryDataSource")
	@ConfigurationProperties(prefix = "spring.datasource.primary")
	public DataSource primaryDataSource() {
		return DataSourceBuilder.create().type(HikariDataSource.class).build();
	}


	@Bean(name = "primaryJdbcTemplate")
	public JdbcTemplate primaryJdbcTemplate(@Qualifier("primaryDataSource") DataSource dataSource) {
		return new JdbcTemplate(dataSource);
	}

	@Bean(name = "secondaryDataSource")
	@Qualifier(value = "secondaryDataSource")
	@ConfigurationProperties(prefix = "spring.datasource.secondary")
	public DataSource secondaryDataSource() {
    /* 这是第一种方法中实例化的对象,也可以用其他的方式实现实例化 */
		// return DataSourceBuilder.create().type(HikariDataSource.class).build();
    /* 这是第二种方法中实例化的对象,也可以用其他的方式实现实例化 */
		return DataSourceBuilder.create().type(JasyptHikariDataSource.class).build();
	}

	@Bean(name = "secondaryJdbcTemplate")
	public NamedParameterJdbcTemplate secondaryJdbcTemplate(@Qualifier("secondaryDataSource") DataSource dataSource) {
		return new NamedParameterJdbcTemplate(dataSource);
	}
}

```

5. 创建测试类,测试程序是否能够获取数据库连接

```java
import java.util.HashMap;
import java.util.List;
import java.util.Map;

import lombok.extern.slf4j.Slf4j;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate;
import org.springframework.test.context.junit4.SpringRunner;

import com.alibaba.fastjson.JSON;

@Slf4j
@SpringBootTest
@RunWith(SpringRunner.class)
public class HikariDataSourceConfigTest {

	@Autowired
	private NamedParameterJdbcTemplate secondaryJdbcTemplate;

  @Test
	public void queryAdGeocodeTest() {
		log.info("secondary data source connection start:");
		String sql = "select 1 from dual";

		List<Map<String, Object>> result = secondaryJdbcTemplate.queryForList(sql,new HashMap<>());
		log.info("secondary data source:\t{}",JSON.toJSONString(result));
	}
	
}
```

* 运行正常即可

## 使用`Druid`数据库连接池(项目连接的是`Oracle`数据库,其他数据库看实际情况修改导入的连接依赖即可)

1. 修改`pom.xml`添加依赖

  ```xml
  <!-- 数据库连接池 -->
  <!-- https://mvnrepository.com/artifact/com.alibaba/druid-spring-boot-starter -->
  <dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>${druid.version}</version>
  </dependency>

  <!-- jdbc -->
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
  </dependency>
  <!-- 本地连接oracle的文件 -->
    <dependency>
    <groupId>com.oracle.ojdbc</groupId>
    <artifactId>ojdbc8</artifactId>
    <version>${ojdbc.version}</version>
  </dependency>
  <!-- jasypt 加密依赖 -->
  <dependency>
    <groupId>com.github.ulisesbocchio</groupId>
    <artifactId>jasypt-spring-boot-starter</artifactId>
    <version>2.1.2</version>
  </dependency>
  ```
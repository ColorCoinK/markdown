---
title:
  - SpringBoot + Durid | Hikari 多数据源 JDBCTemplate
toc: true
layout: SpringBoot
categories:
  - SpringBoot
tags:
  - SpringBoot
  - Durid
  - Hikari
  - JDBCTemplate
date: 2018/07/19 9:55:23
---
SpringBoot + Durid  多数据源 JDBCTemplate
SpringBoot + Hikari  多数据源 JDBCTemplate
<!-- more -->
# SpringBoot 配置多数据源(Druid | Hikari)

> 注释:

Druid：阿里系数据连接池	
Hikari：`SpringBoot` 2.0开始推`HikariCP`，将默认的数据库连接池`tomcat jdbc pool`改为了`hikari`	

## SpringBoot + Druid 配置

## Druid-Jdbctemplate 连接配置 

`jdbctemplate`连接	

1. 增加`pom`依赖	
   
	```xml
	<!-- 本地连接oracle的文件 -->
	<dependency>
		<groupId>com.oracle.ojdbc</groupId>
		<artifactId>ojdbc8</artifactId>
		<version>${ojdbc.version}</version>
	</dependency>

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
	```

2. 设置`application.yml`文件配置	
   
	```yaml
	spring:
		autoconfigure:
			## 多数据源下必须排除掉 DataSourceAutoConfiguration,否则会导致循环依赖报错
			exclude:
			- org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration
		datasource:
			type: com.alibaba.druid.pool.DruidDataSource
			druid:
				## 以`spring.datasources`和`spring.datasource.druid`开头的属性会作为公共配置,注入到每一个数据源
				initial-size: 5
				min-idle: 5 
				max-active: 20
				stat-view-servlet:
					login-username: admin
					login-password: admin
				max-wait: 60000
				time-between-eviction-runs-millis: 60000 ## 配置间隔多久才进行一次检测,检测需要关闭的空闲连接.单位是毫秒
				min-evictable-idle-time-millis: 300000 ## 配置一个连接池中最小生存的时间,单位是毫秒      
				## 配置监控统计拦截的filters,去掉后监控界面SQL无法进行统计,`wall`用于防火墙(https://blog.csdn.net/garyond/article/details/80189939)
				filters: config,stat,wall,log4j
				web-stat-filter:
					exclusions: '*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*'
				## 多数据源的标识,若该属性存在则为多数据源环境,不存在则为但数据源环境
				data-sources: 
					primary:
						url: jdbc:oracle:thin:@//xx:1521/xx
						username: xx
						password: xx
						driverClassName: oracle.jdbc.driver.OracleDriver
					secondary:
						url: jdbc:oracle:thin:@//xx:1521/xx
						username: xx
						password: xx
						driverClassName: oracle.jdbc.driver.OracleDriver
	```

3. Druid 连接池配置类 

	```java
	import javax.sql.DataSource;

	import org.springframework.beans.factory.annotation.Qualifier;
	import org.springframework.boot.context.properties.ConfigurationProperties;
	import org.springframework.context.annotation.Bean;
	import org.springframework.context.annotation.Configuration;
	import org.springframework.context.annotation.Primary;
	import org.springframework.jdbc.core.JdbcTemplate;

	import com.alibaba.druid.pool.DruidDataSource;

	/**
	* 
	* @ClassName: DruidDataScouConfig
	* @Description: 多数据源,集成Druid
	* @author time
	* @date 2018/10/29
	*/
	@Configuration
	public class DruidDataScouConfig {

		@Primary//必需注解，缺少该注解将启动异常.可自定义某个数据源为主数据源
		@Bean(name = "primaryDataSource")
		@Qualifier(value = "primaryDataSource")
		@ConfigurationProperties(prefix = "spring.datasource.druid.data-sources.primary")
		public DataSource primaryDataSource() {
			return new DruidDataSource();
		}

		@Bean(name = "secondaryDataSource")
		@Qualifier(value = "secondaryDataSource")
		@ConfigurationProperties(prefix = "spring.datasource.druid.data-sources.secondary")
		public DataSource secondaryDataSource() {
			return new DruidDataSource();
		}

		@Bean(name = "primaryJdbcTemplate")
		public JdbcTemplate primaryJdbcTemplate(@Qualifier("primaryDataSource") DataSource dataSource) {
			return new JdbcTemplate(dataSource);
		}

		@Bean(name = "secondaryJdbcTemplate")
		public JdbcTemplate secondaryJdbcTemplate(@Qualifier("secondaryDataSource") DataSource dataSource) {
			return new JdbcTemplate(dataSource);
		}
	}
	```

4. 测试数据源是否可用	

	```java
	package com.sanss.config;

	import java.util.List;
	import java.util.Map;

	import org.junit.Test;
	import org.junit.runner.RunWith;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.beans.factory.annotation.Qualifier;
	import org.springframework.boot.test.context.SpringBootTest;
	import org.springframework.jdbc.core.JdbcTemplate;
	import org.springframework.test.context.junit4.SpringRunner;

	import com.alibaba.fastjson.JSON;

	@RunWith(SpringRunner.class)
	@SpringBootTest
	public class DruidDataSourceTest {

		@Autowired
		// @Qualifier("primaryJdbcTemplate")
		private JdbcTemplate primaryJdbcTemplate;

		@Autowired
		// @Qualifier("secondaryJdbcTemplate")//注解可省略
		private JdbcTemplate secondaryJdbcTemplate;

		@Test
		public void TestPrimaryDataSourceConnect() {
			System.err.println("primary data source connection start:");
			String sql = "select 1 from dual";
			List<Map<String, Object>> result = primaryJdbcTemplate.queryForList(sql);
			System.out.println(JSON.toJSONString(result));
		}

		@Test
		public void TestSecondaryDataSourceConnect() {
			System.err.println("secondary data source connection start:");
			String sql = "select 1 from dual";
			List<Map<String, Object>> result = secondaryJdbcTemplate.queryForList(sql);
			System.out.println(JSON.toJSONString(result));
		}

	}
	```

* 注: 连接池配置参数,可以参考<a href='https://mp.weixin.qq.com/s/hueQXuFY7ZLJ1Udysc4F6g'>数据库连接配置策略和实践指南</a>

## Hikari-jdbctemplate 连接配置	

1. 添加`pom.xml`依赖	
   	
	```xml
	<!-- 本地连接oracle的文件 -->
	<dependency>
		<groupId>com.oracle.ojdbc</groupId>
		<artifactId>ojdbc8</artifactId>
		<version>${ojdbc.version}</version>
	</dependency>
	<!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-data-jpa -->
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-data-jpa</artifactId>
	</dependency>
	```

2. 设置`application.yml`多环境配置文件	

```yaml
spring:
  datasource:
    primary: 
      jdbc-url: jdbc:oracle:thin:@//xx:1521/xx
      username: xx
      password: xx
      driver-class-name: oracle.jdbc.driver.OracleDriver
    secondary: 
      jdbc-url: jdbc:oracle:thin:@//xx:1521/xx
      username: xx
      password: xx
      driver-class-name: oracle.jdbc.driver.OracleDriver
```

3. Hikari 数据源配置	

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

/**
 * 
 * @ClassName: HikariDataSourceConfig
 * @Description: Hikari 多数据源配置
 * @author time
 * @date 2018/10/29
 */
@Configuration
public class HikariDataSourceConfig {

	@Primary
	@Bean(name = "primaryDataSource")
	@Qualifier(value = "primaryDataSource")
	@ConfigurationProperties(prefix = "spring.datasource.primary")
	public DataSource primaryDataSource() {
		return DataSourceBuilder.create().type(HikariDataSource.class).build();
	}

	@Bean(name = "secondaryDataSource")
	@Qualifier(value = "secondaryDataSource")
	@ConfigurationProperties(prefix = "spring.datasource.secondary")
	public DataSource secondaryDataSource() {
		return DataSourceBuilder.create().type(HikariDataSource.class).build();
	}

	@Bean(name = "primaryJdbcTemplate")
	public JdbcTemplate primaryJdbcTemplate(@Qualifier("primaryDataSource") DataSource dataSource) {
		return new JdbcTemplate(dataSource);
	}

	@Bean(name = "secondaryJdbcTemplate")
	public JdbcTemplate secondaryJdbcTemplate(@Qualifier("secondaryDataSource") DataSource dataSource) {
		return new JdbcTemplate(dataSource);
	}

}
```

4. 测试数据源是否可用	

```java
import java.util.List;
import java.util.Map;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.test.context.junit4.SpringRunner;

import com.alibaba.fastjson.JSON;

@RunWith(SpringRunner.class)
@SpringBootTest
public class HikariDataSourceTest {

	@Autowired
	private JdbcTemplate primaryJdbcTemplate;

	@Autowired
	private JdbcTemplate secondaryJdbcTemplate;

	@Test
	public void TestPrimaryDataSourceConnect() {
		System.err.println("primary data source connection start:");
		String sql = "select 1 from dual";
		List<Map<String, Object>> result = primaryJdbcTemplate.queryForList(sql);
		System.out.println("primary data source :\t"+JSON.toJSONString(result));
	}

	@Test
	public void TestSecondaryDataSourceConnect() {
		System.err.println("secondary data source connection start:");
		String sql = "select 1 from dual";
		List<Map<String, Object>> result = secondaryJdbcTemplate.queryForList(sql);
		System.out.println("secondary data source :\t"+JSON.toJSONString(result));
	}

}
```

## 配置 NamedParameterJdbcTemplate

1. `pom.xml`依赖、`applicaiton.yml`数据源配置文件内容与`jdbctemplate`配置一致

2. 需要修改配置类返回的实例为`NamedParameterJdbcTemplate`

```java
	@Bean(name = "secondaryDataSource")
	@Qualifier(value = "secondaryDataSource")
	@ConfigurationProperties(prefix = "spring.datasource.secondary")
	public DataSource secondaryDataSource() {
		return DataSourceBuilder.create().type(HikariDataSource.class).build();
	}

	@Bean(name = "secondaryJdbcTemplate")
	public NamedParameterJdbcTemplate secondaryJdbcTemplate(
			@Qualifier("secondaryDataSource") DataSource dataSource) {
		return new NamedParameterJdbcTemplate(dataSource);
	}
```

3. 测试类

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class HikariDataSourceConfigTest {

	@Autowired
	private NamedParameterJdbcTemplate secondaryJdbcTemplate;

	@Test
	public void TestNamedParameterJdbcTemplate() {
		System.err.println("namedParameterJdbcTemplate data source connection start:");
		String sql = "select 1 from dual";

		int result = secondaryJdbcTemplate.queryForObject(sql, new HashMap<>(), Integer.class);
		System.out.println("TestNamedParameterJdbcTemplate data source:\t" + result);
	}
}

```
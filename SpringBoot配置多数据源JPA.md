---
title: SpringBoot + Hikari +JPA 多数据源
toc: true
layout: SpringBoot
categories:
  - SpringBoot
tags:
  - SpringBoot
  - Hikari
  - JPA
date: 2018/07/19 9:55:23
---
{{title}}
<!-- more -->
# <h3>SpringBoot2.0 Hikari 多数据源 —— JPA </h3>

`SpringBoot` 连接多数据源使用 `JPA` 查询应用记录.

## `pom.xml`依赖配置 	
```
<!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-data-jpa -->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```
注：`SpringBoot` 2.0默认的数据连接池为`Hikari`，故不需要添加 `Hikari` 依赖	

## 多数据源配置		

> 1. `primaryDataSource` 配置		

```java
package com.learnning.config;

import javax.persistence.EntityManager;
import javax.sql.DataSource;

import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.boot.orm.jpa.EntityManagerFactoryBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;
import org.springframework.orm.jpa.JpaTransactionManager;
import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.annotation.EnableTransactionManagement;

import com.zaxxer.hikari.HikariDataSource;

/**
 * 
 * @ClassName: HikariDataSourceConfig
 * @Description: Hikari 多数据源配置(第一数据源配置)
 * @author time
 * @date 2018/10/29
 * {@link {@link http://blog.didispace.com/springbootmultidatasource/}}
 */
@Configuration
@EnableTransactionManagement
@EnableJpaRepositories(entityManagerFactoryRef = "primaryEntityManagerFactory", //
		transactionManagerRef = "primaryTransactionManager", //
		basePackages = { "com.learnning.domain.p" }) // 设置Repository所在位置
public class PrimaryConfig {

	@Primary
	@Bean(name = "primaryDataSource")
	@Qualifier(value = "primaryDataSource")
	@ConfigurationProperties(prefix = "spring.datasource.primary")
	public DataSource primaryDataSource() {
		return DataSourceBuilder.create().type(HikariDataSource.class).build();
	}

	@Primary
	@Bean(name = "primaryEntityManager")
	public EntityManager entityManager(EntityManagerFactoryBuilder builder) {
		return primaryEntityManagerFactory(builder).getObject().createEntityManager();
	}

	@Primary
	@Bean(name = "primaryEntityManagerFactory")
	public LocalContainerEntityManagerFactoryBean primaryEntityManagerFactory(EntityManagerFactoryBuilder builder) {
		return builder//
				.dataSource(primaryDataSource()) //
				.packages("com.learnning.domain.p")//
				.persistenceUnit("primaryPersistenceUnit")//
				.build();
	}

	@Primary
	@Bean(name = "primaryTransactionManager")
	public PlatformTransactionManager transactionManager(EntityManagerFactoryBuilder builder) {
		return new JpaTransactionManager(primaryEntityManagerFactory(builder).getObject());
	}

}
```

>  2. `sencondaryDataSource` 配置		

```java
package com.learnning.config;

import javax.persistence.EntityManager;
import javax.sql.DataSource;

import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.boot.orm.jpa.EntityManagerFactoryBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;
import org.springframework.orm.jpa.JpaTransactionManager;
import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.annotation.EnableTransactionManagement;

import com.zaxxer.hikari.HikariDataSource;

/**
 * 
 * @ClassName: HikariDataSourceConfig
 * @Description: Hikari 多数据源配置(第一数据源配置)
 * @author time
 * @date 2018/10/29
 * {@link {@link http://blog.didispace.com/springbootmultidatasource/}}
 */
@Configuration
@EnableTransactionManagement
@EnableJpaRepositories(entityManagerFactoryRef = "secondaryEntityManagerFactory", //
		transactionManagerRef = "secondaryTransactionManager", //
		basePackages = { "com.learnning.domain.s" }) // 设置Repository所在位置
public class SecondaryConfig {

	@Bean(name = "sencondaryDataSource")
	@Qualifier(value = "sencondaryDataSource")
	@ConfigurationProperties(prefix = "spring.datasource.secondary")
	public DataSource sencondaryDataSource() {
		return DataSourceBuilder.create().type(HikariDataSource.class).build();
	}

	@Bean(name = "secondaryEntityManager")
	public EntityManager entityManager(EntityManagerFactoryBuilder builder) {
		return secondaryEntityManagerFactory(builder).getObject().createEntityManager();
	}

	@Bean(name = "secondaryEntityManagerFactory")
	public LocalContainerEntityManagerFactoryBean secondaryEntityManagerFactory(EntityManagerFactoryBuilder builder) {
		return builder//
				.dataSource(sencondaryDataSource()) //
				.packages("com.learnning.domain.s")//
				.persistenceUnit("secondaryPersistenceUnit")//
				.build();
	}

	@Bean(name = "secondaryTransactionManager")
	PlatformTransactionManager transactionManager(EntityManagerFactoryBuilder builder) {
		return new JpaTransactionManager(secondaryEntityManagerFactory(builder).getObject());
	}

}
```

## `JPA` 使用多数据源查询		

**划重点：**		

1. `JPA`的`entity`与table因为有映射关系，所以实体类路径需要映射到对应的`DataSource`中	

2. `JPA`查询接口最好也根据不同数据源区分开	

> `primary`数据源配置	

* `primary`数据源实体类	

```java
package com.learnning.domain.p;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.Table;

@Entity
@Table(name = "t_account")
public class User {

	@Id
	@GeneratedValue
	private Long account_id;

	private String user_name;

	private Integer validity;

	public Long getAccount_id() {
		return account_id;
	}

	public void setAccount_id(Long account_id) {
		this.account_id = account_id;
	}


	public String getUser_name() {
		return user_name;
	}

	public void setUser_name(String user_name) {
		this.user_name = user_name;
	}

	public Integer getValidity() {
		return validity;
	}

	public void setValidity(Integer validity) {
		this.validity = validity;
	}

	@Override
	public String toString() {
		return "{\"account_id\":\"" + account_id + "\",\"user_name\":\"" + user_name + "\",\"validity\":\"" + validity
				+ "\"}";
	}
}
```

* `primary` 数据源`Repository`
  	
```java
package com.learnning.domain.p.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;

import com.learnning.domain.p.User;

public interface UserRepository extends JpaRepository<User, Long> {

	/**
	 * 
	 * @Title: findByAccount_id
	 * @Description: 查询用户 User 信息
	 * 	nativeQuery：指定为原生SQL查询
	 * @param account_id
	 * @return
	 * User
	 */
	/// 模糊匹配
	@Query(value = "select account_id,user_name,validity from t_account where account_id = ?", nativeQuery = true)
	User findByAccount_id(Long account_id);

}
```

> `secondary` 数据源配置	

* `secondary` 实体类 
  
```java
package com.learnning.domain.s;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.Table;

@Entity
@Table(name = "t_user")
public class PopUser {

	@Id
	@GeneratedValue
	private Long pop_id;

	private String pop_name;

	private String ip_start;

	private String ip_end;

	private String netmask;

	private String description;

	private String start_num;

	private String end_num;

	private String platform;

	private Integer ipgap;

	private Integer pop_type;

	public Long getPop_id() {
		return pop_id;
	}

	public void setPop_id(Long pop_id) {
		this.pop_id = pop_id;
	}

	public String getPop_name() {
		return pop_name;
	}

	public void setPop_name(String pop_name) {
		this.pop_name = pop_name;
	}

	public String getIp_start() {
		return ip_start;
	}

	public void setIp_start(String ip_start) {
		this.ip_start = ip_start;
	}

	public String getIp_end() {
		return ip_end;
	}

	public void setIp_end(String ip_end) {
		this.ip_end = ip_end;
	}

	public String getNetmask() {
		return netmask;
	}

	public void setNetmask(String netmask) {
		this.netmask = netmask;
	}

	public String getDescription() {
		return description;
	}

	public void setDescription(String description) {
		this.description = description;
	}

	public String getStart_num() {
		return start_num;
	}

	public void setStart_num(String start_num) {
		this.start_num = start_num;
	}

	public String getEnd_num() {
		return end_num;
	}

	public void setEnd_num(String end_num) {
		this.end_num = end_num;
	}

	public String getPlatform() {
		return platform;
	}

	public void setPlatform(String platform) {
		this.platform = platform;
	}

	public Integer getIpgap() {
		return ipgap;
	}

	public void setIpgap(Integer ipgap) {
		this.ipgap = ipgap;
	}

	public Integer getPop_type() {
		return pop_type;
	}

	public void setPop_type(Integer pop_type) {
		this.pop_type = pop_type;
	}

	@Override
	public String toString() {
		return "{\"pop_id\":\"" + pop_id + "\",\"pop_name\":\"" + pop_name + "\",\"ip_start\":\"" + ip_start
				+ "\",\"ip_end\":\"" + ip_end + "\",\"netmask\":\"" + netmask + "\",\"description\":\"" + description
				+ "\",\"start_num\":\"" + start_num + "\",\"end_num\":\"" + end_num + "\",\"platform\":\"" + platform
				+ "\",\"ipgap\":\"" + ipgap + "\",\"pop_type\":\"" + pop_type + "\"}";
	}

}
```

* `secondary` 数据源 `Repository` 

```java
package com.learnning.domain.s.repository;

import java.util.List;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;

import com.learnning.domain.s.PopUser;

public interface PopUserRepository extends JpaRepository<PopUser, Long> {

	@Query(value = "select pop_name,pop_id,ip_start,ip_end,netmask,description,start_num,end_num,platform,ipgap,pop_type from t_user where rownum<= 10", nativeQuery = true)
	List<PopUser> queryPopUserPage1();

}
```

## 测试类	

```java
@Autowired
private UserRepository userRepository;

@Autowired
private PopUserRepository popUserRepository;

@Test
public void testPrimarySession() {
	User users = userRepository.findByAccount_id(1L);
	System.out.println(users.toString());

	List<PopUser> page = popUserRepository.queryPopUserPage1();
	System.out.println(page.toString());

}
```

**注：** `SpringBoot`官方建议使用构造函数的方式注入依赖，上述测试代码为在测试类中使用.

使用构造函数注入如下：

```java
private UserRepository userRepository;

private PopUserRepository popUserRepository;

@Autowired
public MyErrorController(UserRepository userRepository, PopUserRepository popUserRepository) {
	this.userRepository = userRepository;
	this.popUserRepository = popUserRepository;
}
```
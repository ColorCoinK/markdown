---
title: SpringBoot多数据源——impala连接
toc: true
layout: springboot
type: springboot
categories:
  - SpringBoot
tags:
  - SpringBoot
  - Java
date: 2019-04-02 14:30:54
---
SpringBoot 2.0 + Impala 多数据源 JDBCTemplate
SpringBoot 2.0 + Impala 多数据源 Named
<!-- more -->

# SpringBoot 配置多数据源(Hikari连接池)连接Impala

> 注释:

Impala:

## SpringBoot使用默认的Hikari连接池,连接Impala

1. 增加依赖

```xml
<!--数据库连接池-->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>

<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-configuration-processor</artifactId>
  <optional>true</optional>
</dependency>

<!-- Impala start -->
<!-- 有可能下载不到该Jar, 可以到该地址下载相应版本：https://www.cloudera.com/downloads/connectors/impala/jdbc/2-6-3.html -->
<!-- https://mvnrepository.com/artifact/com.cloudera/ImpalaJDBC41 -->
<dependency>
  <groupId>com.cloudera</groupId>
  <artifactId>ImpalaJDBC41</artifactId>
  <version>${impala.version}</version>
</dependency>
<!-- Impala end -->
```

2. 配置`application.yml`文件

- 该配置文件为多环境配置,可以参考 <a href="../SpringBoot多环境配置">SpringBoot多环境配置</a>

```yaml
server:
  port: 7001
spring:
  profiles:
    active: dev
  servlet:
    multipart:
      enabled: true
  mvc:
    view:
      prefix: /
      suffix: .html
    favicon:
      enabled: false
logback:
  name: template

---
# developer enviroment
spring:
  profiles: dev
  datasource:
    impala: 
      driver-class-name: com.cloudera.impala.jdbc41.Driver
      jdbc-url: jdbc:impala://xx:21050/default
    oracle:
      first:
        driver-class-name: oracle.jdbc.OracleDriver
        jdbc-url: jdbc:oracle:thin:@//xx:1521/pdb1.us.oracle.com
        username: xx
        password: xx
---
# developer enviroment
spring:
  profiles: test
  datasource:
    impala: 
      driver-class-name: com.cloudera.impala.jdbc41.Driver
      jdbc-url: jdbc:impala://xx:21050/default
    oracle:
      first:
        driver-class-name: oracle.jdbc.OracleDriver
        jdbc-url: jdbc:oracle:thin:@//xx:1521/pdb1.us.oracle.com
        username: xx
        password: xx
---
# developer enviroment
spring:
  profiles: prod
  datasource:
    impala: 
      driver-class-name: com.cloudera.impala.jdbc41.Driver
      jdbc-url: jdbc:impala://xx:21050/default
    oracle:
      first:
        driver-class-name: oracle.jdbc.OracleDriver
        jdbc-url: jdbc:oracle:thin:@//xx:1521/pdb1.us.oracle.com
        username: xx
        password: xx
```

3. 多数据源连接配置类

* 项目目录结构

```
src
├───main
│   ├───java
│   │   └───com
│   │       └───template
│   │           ├───common      # 公共部分
│   │           ├───config      # 配置、数据源
│   │           ├───domain      # DO、DTO、VO
│   │           ├───repository  # 数据库访问层
│   │           ├───service     # 逻辑层
│   │           │   └───impl    # 逻辑具体实现
│   │           ├───util        # 工具类
│   │           └───web         # api 接口
│   └───resources
│       ├───static              # 静态资源文件
│       │   └───js              # 页面依赖的javascript 文件
│       └───templates           # 页面模板文件
└───test
    └───java
        └───com
            └───template
                ├───config
                └───repository
```

* 多数据源配置类

```java
package com.template.config;

import javax.sql.DataSource;

import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate;

import com.zaxxer.hikari.HikariDataSource;

@Configuration
public class HikariDataSourceConfig {

    @Primary
    @Bean(name = "impalaDataSource")
    @Qualifier(value = "impalaDataSource")
    @ConfigurationProperties(prefix = "spring.datasource.impala")
    public DataSource primaryDataSource() {
        return DataSourceBuilder.create().type(HikariDataSource.class).build();
    }

    @Bean(name = "jdbcTemplateImpala")
    public JdbcTemplate jdbcTemplateImpala(@Qualifier("impalaDataSource") DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }

    @Bean(name = "namedJdbcTemplateImpala")
    public NamedParameterJdbcTemplate namedJdbcTemplateImpala(
            @Qualifier("impalaDataSource") DataSource dataSource) {
        return new NamedParameterJdbcTemplate(dataSource);
    }

    @Bean(name = "secondOracleDataSource")
    @Qualifier(value = "secondOracleDataSource")
    @ConfigurationProperties(prefix = "spring.datasource.oracle.first")
    public DataSource firstOracleDataSource() {
        return DataSourceBuilder.create().type(HikariDataSource.class).build();
    }

    @Bean(name = "jdbcTemplateOracle1")
    public JdbcTemplate jdbcTemplateOracle1(
            @Qualifier("secondOracleDataSource") DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }

}
```

4. 测试类

```java
package com.template.config;


import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.test.context.junit4.SpringRunner;

import lombok.extern.slf4j.Slf4j;

@Slf4j
@SpringBootTest
@RunWith(SpringRunner.class)
public class HikariDataSourceConfigTest {

    @Autowired
    private JdbcTemplate jdbcTemplateImpala;

    @Test
    public void testImpalaJdbcTemplate() {
        log.info("impala jdbctemplate connection start:");
        String sql = "select count(1) from t_info_iptvorder";
        Integer count = jdbcTemplateImpala.queryForObject(sql, Integer.class);
        log.info("impalaJdbcTemplate query result :\t" + count);

    }

}
```
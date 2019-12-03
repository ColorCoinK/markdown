---
title: SpringBoot Log
toc: true
layout: SpringBoot
categories:
  - SpringBoot
tags:
  - SpringBoot
  - Apply
date: 2018/07/19 9:55:23
---
{{title}}
<!-- more -->
> `Transactional`事务回滚		

```java
//方法上增加注解
@Transactional(rollbackOn = Exception.class)
public void example() {
	try {

	} catch (Exception e) {
		// 设置异常时执行回滚
		TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();
	}
}
```

> 配置类序列化失败		

问题描述：
	
	将配置类使用 JackSon 序列化时，出现序列化异常

> `SpringBoot`请求跨域	

```java
package com.learnning.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;
import org.springframework.web.filter.CorsFilter;

/**
 * 
 * @ClassName: CustomCORSConfiguration
 * @Description: 配置跨域请求访问失败的问题
 * @author time
 * @date 2018/11/21
 */
@Configuration
public class CustomCORSConfiguration {

	private CorsConfiguration buildConfig() {
		CorsConfiguration corsConfiguration = new CorsConfiguration();
		corsConfiguration.addAllowedOrigin("*");
		corsConfiguration.addAllowedHeader("*");
		corsConfiguration.addAllowedMethod("*");
		return corsConfiguration;
	}

	@Bean
	public CorsFilter corsFilter() {
		UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
		source.registerCorsConfiguration("/**", buildConfig());
		return new CorsFilter(source);
	}

}
```

* 注: 引用链接: <a href="http://www.itmuch.com/work/cors/" target="_blank">周立解决方案</a>		

> `SpringBoot JPA`查询传参	

1. 使用占位符`?`	
   
```java
@Query(value = "select account_id,user_name,validity from t_account where account_id = ?", nativeQuery = true)
User findByAccount_id(Long account_id);
```

2. 使用命名化参数`:name` 

```java
@Query(value = "select account_id,user_name,validity from t_account where account_id =:id", nativeQuery = true)
User findByAccount_id(@Param("id") Long account_id);
```

* 注: 参考<a href="http://www.spring4all.com/article/464" target="_blank">Spring Data JPA系列: 使用@Query注解（Using @Query）</a>
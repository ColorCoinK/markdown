---
title: SpringBoot + Redis 自定义主键增长策略
toc: true
layout: SpringBoot
categories:
  - SpringBoot
tags:
  - SpringBoot
  - Redis
date: 2018/07/19 9:55:23
---
{{title}}
<!-- more -->
# <h3>SpringBoot 使用 Redis 生成yyyyMMdd + 0001 格式id</h3>

> 开发环境 

* 当前开发环境:	`SpringBoot + redis`，应该可以用于其他同样使用`Redis`的环境生成唯一`id`环境(未曾亲自实践)
* 适用场景: 单个服务的唯一`id`生成工具类	

> 如果之前未引入`Redis`则需要增加依赖	

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

> 在`.properties`文件中增加`Redis`配置	

```properties
# REDIS的配置信息
# Redis数据库索引（默认为0）
spring.redis.database=0  
# 指定Redis服务器地址，多环境/注释实现(本文有参考链接)
spring.redis.host=127.0.0.1
# Redis服务器连接端口
spring.redis.port=6379
# 用户缓存超时时间
spring.redis.expireTime=1800
# Redis服务器连接密码（默认为空）
#spring.redis.password=foobared
# 连接池最大连接数（使用负值表示没有限制）
spring.redis.jedis.pool.max-active=8  
# 连接池最大阻塞等待时间（使用负值表示没有限制）
spring.redis.jedis.pool.max-wait=-1  
# 连接池中的最大空闲连接
spring.redis.jedis.pool.max-idle=8  
# 连接池中的最小空闲连接
spring.redis.jedis.pool.min-idle=0  
# 连接超时时间（毫秒）
spring.redis.timeout=12000 
```

* 注: SpringBoot多环境配置[^多环境配置] 	

> 工具类代码片段	

```java
import java.text.DateFormat;
import java.text.SimpleDateFormat;
import java.util.Calendar; 
import java.util.Date;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.support.atomic.RedisAtomicLong;
import org.springframework.stereotype.Service;

/**
 * 
 * @ClassName: RedisGeneratorCode
 * @Description: 生成日期开头加0001，0002，0003的工具类
 * @author time
 * @date 2018/10/09
 * {@link https://www.cnblogs.com/jbml-154312/p/7490810.html}
 */
@Service
public class RedisGeneratorCode {

	private static final Logger logger = LoggerFactory.getLogger(RedisGeneratorCode.class);

	private RedisTemplate<String, Object> redisTemplate;

	@Autowired
	public RedisGeneratorCode(RedisTemplate<String, Object> redisTemplate) {
		this.redisTemplate = redisTemplate;
	}

	/**
	 * 
	 * @Title: generateCode
	 * @Description: 生成自定义前缀的类似 HTG201810120001格式的自增数
	 * @param key
	 * @param prefix
	 * @param hasExpire
	 * @param minLength
	 * @return
	 * String
	 */
	public String generateCode(String key, String prefix, boolean hasExpire, Integer minLength) {
		return this.createGenerateCode(key, prefix, hasExpire, minLength);
	}

	/**
	 * 
	 * @Title: generateCode
	 * @Description: 生成 类似 201810120001格式的自增数
	 * @param key
	 * @param hasExpire
	 * @param minLength
	 * @return
	 * String
	 */
	public String generateCode(String key, boolean hasExpire, Integer minLength) {
		return this.createGenerateCode(key, "", hasExpire, minLength);
	}

	/**
	 * 
	 * @Title: generateCode
	 * @Description: 组装符合自己规则的id并设置过期时间
	 * @param key	redis中的key值
	 * @param prefix	最后编码的前缀
	 * @param hasExpire	redis 是否使用过期时间设置自增id
	 * @param minLength	redis生成的自增id的最小长度，如果小于这个长度前面补0
	 * @return
	 * String
	 */
	public String createGenerateCode(String key, String prefix, boolean hasExpire, Integer minLength) {
		try {
			Date date = null;
			Long id = null;
			Calendar calendar = Calendar.getInstance();
			if (hasExpire) {
				calendar.set(Calendar.HOUR_OF_DAY, 23);
				calendar.set(Calendar.MINUTE, 59);
				calendar.set(Calendar.SECOND, 59);
				calendar.set(Calendar.MILLISECOND, 9999);
				date = calendar.getTime();
			} else {
				calendar.set(Calendar.MINUTE, calendar.get(Calendar.MINUTE) + 10);
				date = calendar.getTime();
			}
			id = this.generateId(key, date);
			if (id != null) {
				return this.format(id, prefix, minLength);
			}
		} catch (Exception e) {
			logger.info("error --> redis 生成自增id出现异常");
			logger.error(e.getMessage(), e);
		}
		return null;
	}

	/**
	 * 
	 * @Title: generateId
	 * @Description: 使用RedisAtomicLong自增
	 * @param key
	 * @param date
	 * @return
	 * Long
	 */
	private Long generateId(String key, Date date) {
		RedisAtomicLong counter = new RedisAtomicLong(key, redisTemplate.getConnectionFactory());
		// 通过key获取自增并设定过期时间
		counter.expireAt(date);
		return counter.incrementAndGet();
	}

	/**
	 * 
	 * @Title: format
	 * @Description: 获取 redis 自增后，生成自定义格式的id
	 * @param id	redis 获取的 id值
	 * @param prefix	自定义前缀
	 * @param minLength	生成数的长度，不满足时 0 补齐
	 * @return
	 * String
	 */
	private String format(Long id, String prefix, Integer minLength) {

		// 拼接的字符串
		StringBuffer sb = new StringBuffer();
		// 当前日期
		Date date = new Date();
		// 自定义前缀
		sb.append(prefix);
		if (date != null) {
			DateFormat df = new SimpleDateFormat("yyyyMMdd");
			sb.append(df.format(date));
		}

		/* 对不满足长度的id值,使用0补齐 */
		// redis 生成的id值
		String strId = String.valueOf(id);
		// redis 生成id 的长度
		int length = strId.length();
		if (length < minLength) {
			for (int i = 0; i < minLength - length; i++) {
				sb.append("0");
			}
			sb.append(strId);
		} else {
			sb.append(strId);
		}
		return sb.toString();
	}

}

```

> 测试工具类

```java
@Autowired
private RedisGeneratorCode redisGeneratorCode;

@Test
public void createGenerateCodeTest() {
	// String code = primaryGenerater.generateCode("worksheet_id", "", true, 4);
	boolean flag = true;
	String code = null;
	int count = 0;
	while (flag) {
		code = redisGeneratorCode.generateCode("worksheet_id", "", true, 4);
		flag = code == null ? true : false;
	}
	System.out.println(code);
}
```


> 参考链接:	

1. <a href="https://www.cnblogs.com/jbml-154312/p/7490810.html" target="_blank">java生成自增流水号，并从每月第一天重新清零计数将业务流水号添加到数据库(原创)</a>
2. <a href="https://blog.csdn.net/UnknownZYB/article/details/78334250?locationNum=5&fps=1" target="_blank">使用redis生成全局唯一id</a>
3. <a href="https://www.cnblogs.com/haitao-xie/p/6323423.html" target="_blank">SpringBoot集成Redis</a>

> 脚注	

[^多环境配置]: <a href="../SprinBoot多环境配置/#more" target="_blank">参考另一篇博客`SpringBoot`多环境配置</a>
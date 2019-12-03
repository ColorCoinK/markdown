---
title: Redis生成唯一主键
toc: true
layout: blog
categories:
  - blog
  - Java
tags:
  - Blog
date: 2019-04-24 09:11:17
---
使用Redis生成`自定义前缀/日期格式`自增的唯一主键,例如`HTG201810120001`、`201810120001`等自增数。同时每天的零点重新开始生成个位为`1`的主键
<!-- more -->

* 参考博客: https://www.cnblogs.com/jbml-154312/p/7490810.html

> 前提:

  * 需要连接到`Redis`

# 工具类代码

```java
package com.sanss.city.util;

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
 * @author Dew
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
	 * @param key redis中的key值
	 * @param prefix  最后编码的前缀
	 * @param hasExpire redis 是否使用过期时间设置自增id
	 * @param minLength redis 生成的自增id的最小长度，如果小于这个长度前面补0
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
	 * @param key redis 中的key值
	 * @param hasExpire redis 是否使用过期时间设置自增id
	 * @param minLength redis 生成的自增id的最小长度，如果小于这个长度前面补0
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
	 * @param key redis中的key值
	 * @param date 过期时间
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

# 方法中调用

```java
/**
  * 
  * @Title: getGeneratorId
  * @Description: 获取 yyyyMMdd0001格式,的自增工单id，并每天00:23:59 初始化化为0001
  * @return
  * Long
  */
private Long getGeneratorId() {
  boolean flag = true;
  String code = null;
  int count = 0;
  while (flag) {
    System.err.println(++count);
    code = redisGeneratorCode.generateCode("worksheet_id", "", true, 4);
    flag = code == null ? true : false;
  }
  return Long.parseLong(code);
}
```
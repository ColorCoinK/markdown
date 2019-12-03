---
title: 遍历 Java 实体类属性及值	
toc: true
layout: blog
categories:
  - blog
  - Java
tags:
  - Java
  - Entity
date: 2019-02-28 11:00:13
---
{{title}}
<!-- more -->
# 遍历`Java`实体类属性及值	

> 核心代码	

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

private static final Logger logger = LoggerFactory.getLogger(HcodeRepository.class);

/**
 * 
 * @Title: getPostParams
 * @Description: 将实体类clazz的属性转换为url参数
 * @param clazz	参数实体类
 * @return
 * String
 */
private String getPostParams(Object clazz) {
	Field[] fields = clazz.getClass().getDeclaredFields();

	StringBuilder requestURL = new StringBuilder();
	try {
		boolean flag = true;
		String property, value;
		for (int i = 0; i < fields.length; i++) {
			Field field = fields[i];
			// 允许访问私有变量
			field.setAccessible(true);

			// 属性名
			property = field.getName();
			// 属性值
			value = field.get(clazz).toString();

			System.out.println(property+":"+value);
		}
	} catch (Exception e) {
		logger.error("用户播放轨迹查询Qos 日志失败,参数为：" + clazz.toString());
	}
	return requestURL.toString();
}
```

> 示例	

将查询参数封装为`url`
```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

private static final Logger logger = LoggerFactory.getLogger(HcodeRepository.class);
/**
 * 
 * @Title: getPostParams
 * @Description: 将实体类clazz的属性转换为url参数
 * @param clazz	参数实体类
 * @return
 * String
 */
private String getPostParams(Object clazz) {
	// 遍历属性类、属性值
	Field[] fields = clazz.getClass().getDeclaredFields();

	StringBuilder requestURL = new StringBuilder();
	try {
		boolean flag = true;
		String property, value;
		for (int i = 0; i < fields.length; i++) {
			Field field = fields[i];
			// 允许访问私有变量
			field.setAccessible(true);

			// 属性名
			property = field.getName();
			// 属性值
			value = field.get(clazz).toString();

			String params = property + "=" + value;
			if (flag) {
				requestURL.append(params);
				flag = false;
			} else {
				requestURL.append("&" + params);
			}
		}
	} catch (Exception e) {
		logger.error("URL参数为：" + clazz.toString());
	}
	return requestURL.toString();
}
```


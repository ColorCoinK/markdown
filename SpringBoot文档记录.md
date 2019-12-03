---
title: SpringBoot 在线文档地址
toc: true
layout: SpringBoot
categories:
  - SpringBoot
tags:
  - SpringBoot
  - Document
date: 2018/07/19 9:55:23
---
{{title}}
<!-- more -->
# SpringBoot 在线文档地址		

<a href="https://docs.spring.io/spring-boot/docs/1.5.14.RELEASE/reference/htmlsingle/" target="_blank">英文版</a>	
<a href="">中文版</a>

## 15.1 在`main`函数类中使用`@Import`注解	

```document
You need not put all your @Configuration into a single class. The @Import annotation can be used to import additional configuration classes. Alternatively, you can use @ComponentScan to automatically pick up all Spring components, including @Configuration classes.
```
释义：
  您无需将所有@Configuration放入单个类中。 @Import注释可用于导入其他配置类。或者，您可以使用@ComponentScan自动获取所有Spring组件，包括@Configuration类。 		

## 24.56 使用`Java`类加载 配置文件参数值		

> 配置类	

```java
	@ConfigurationProperties("foo")
	public class FooProperties {

	    private final List<MyPojo> list = new ArrayList<>();

	    public List<MyPojo> getList() {
	        return this.list;
	    }

	}	
```	
* 注: 推荐使用构造函数的方式注入属性值

> 配置文件 	

```yml
foo:
  list:
    - name: my name
      description: my description
---
spring:
  profiles: dev
foo:
  list:
    - name: my another name
```

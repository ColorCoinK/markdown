---
title: SpringBoot配置文件详解
toc: true
layout: springboot
type: springboot
categories:
  - SpringBoot
tags:
  - SpringBoot
  - Java
date: 2019-03-04 09:48:42
---
SpringBoot自定义配置文件内容获取
<!-- more -->

# `SpringBoot`配置文件内容获取

> 前提

1. 添加以下依赖

```xml
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
  </dependency>
```

## 新建实体类获取`application.yml/applicaiton.properties`配置文件内容

1. `@Component`、`@ConfigurationProperties` 注解

> <h5>`application.yml`配置文件</h5>

```yaml
server:
  port: 9099
user:
  name: markdown
  age: 10
```

> <h5>实体类</h5>

```java
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Component
@ConfigurationProperties(prefix = "user")
public class UserConfig implements Serializable {

	private String name;

	private String age;

  /** 省略get/set 方法 */
}
```

> <h5>测试类</h5>

* 测试是否能够正确获取配置文件内容

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class UserConfigTest {

	@Autowired
	private UserConfig properties;

	@Test
	public void propertiesTest() {
		System.out.println(properties.name() + "\t" + properties.getAge());
	}
}
```

2. `@Component`、`@Value` 注解

* `application.yml/application.properties` 文件内容不变

> <h5>实体类</h5>

```java
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Component
public class User2Config implements Serializable {

	@Value(value = "${user.url}")
	private String name;

	@Value(value = "${user.age}")
	private String age;

  /** 省略get/set 方法 */
}
```

> <h5>测试类</h5>

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class User2ConfigTest {

	@Autowired
	private User2Config properties;

	@Test
	public void propertiesTest() {
		System.out.println(properties.name() + "\t" + properties.getAge());
	}
}
```

## 创建实体类获取自定义(`xx.properties`)配置文件内容

  * 注: 自定义配置文件最好还是使用`.properties`格式的文件,注解方式还不支持手动加载`.yml`格式文件的功能

1. `@Configuration`注解替换`@Component`注解,`@PropertySource("classpath:xx.properties")`

> <h5>创建自定义配置文件`my.properties`</h5>

```properties
# 高德——地理编码
geo.url= http://restapi.amap.com/v3/geocode/geo
geo.key= 11111
```

> <h5>实体类</h5>

```java
@Configuration
@PropertySource("classpath:geocode.properties")
public class GeocodingProperties {

	@Value(value = "${geo.url}")
	private String url;

	@Value(value = "${geo.key}")
	private String name;

  /*省略get/set方法*/
}
```

> <h5>测试类</h5>

```java
	@RunWith(SpringRunner.class)
	@SpringBootTest
	public class GeoCodingUtilTest {

		@Autowired
		private GeocodingProperties properties;

		@Test
		public void propertiesTest() {
			System.out.println(properties.getKey() + "\t" + properties.getUrl());
		}
	}
```

* 配置文件可创建在`resource`中的文件夹下,使用相对根目录的文件夹即可(默认`resources/`为根目录)

## `@Configuration`、`@PropertySource`、`@ConfigurationProperties`

* 注: 使用`@configuration`与`@ConfigurationProperties`注解时,需要在启动类(`Application.java`)中增加`@EnableConfigurationProperties(PhoneProperties.class)`注解。

> 配置文件

* 配置文件路径`resources/properties/phone.properties")`

```java
phone.model= iPhone 8 Plus
phone.price= 5266
phone.system= iOS 12.2.x
```

> 启动类	

```java
@SpringBootApplication
@EnableConfigurationProperties(PhoneProperties.class)
public class Application {

	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}

}
```

> 配置文件实体类

```java
@Configuration
@PropertySource(value = "classpath:properties/phone.properties")
@ConfigurationProperties(prefix = "phone")
public class PhoneProperties {

	private String model;

	private Double price;

	private String system;

	/*省略get/set、toString()方法*/
}
```

> 测试类

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class PhonePropertiesTest {

	@Autowired
	private PhoneProperties phoneProperties;

	@Test
	public void phonePropertiesTest() {
		System.out.println(phoneProperties.toString());
	}

}
```

## 使用`Environment`方法获取配置文件的值


> 注解说明  

* `@Component`

	1. 将普通的`pojo`实例化到`Spring`容器中,相当于`xml`配置文件中的`<bean id="" class=""/>`;

	2. 泛指组件,当组件不好归类的的时候,可以使用这个组件。

* `@Configutration`

	1. 用于定义配置类;

	2. `@Configuration` 中所有带`@Bean`注解的方法都会被动态代理,因此调用该方法返回的都是同一个实例。

	3. `@Component` 与 `@Component` 区别详细介绍,<a href="https://blog.csdn.net/isea533/article/details/78072133" target="_blank">`Spring @Configuration 和 @Component`区别</a>

* `@PropertySource`

	1. 通过注解的方式将`properties`配置文件中的值存储到`Spring`的`Enviroment`中,`Environment`接口提供方法去读取配置文件中的值,参数时`properties`文件中定义的`key`值。

* `@ConfigurationProperties`

	1. 将同类的配置信息封装成实体类;

	2. 与`@ConfigurationProperties`一起使用时,需要在启动类增加`@EnableConfigurationProperties`注解并制定该实体类为配置类 或 使用`@Component`注解替代`@Configuration`注解。

* `@Value`

	1. `@value`属性名,在属性名上添加该注解;

	2. 默认读取的配置文件是`application.yml/application.properties`,可在`@RestController/@Controller`中获取配置。

## 常见问题

### <h5>在非`Controller` 中无法获取配置类 或 注入相应注解</h5>

* 在工具类中,获取`RestTemplate`注解以及自定义配置文件为例

```java
// Point 1: 使用类注解`@Conponent`
@Component
public class GeoCodingUtil {

	@Autowired
	private RestTemplate restTemplate;

	@Autowired
	private GeocodingProperties properties;

	// Point 2: 将本类设置为属性
	private static GeoCodingUtil geoCodingUtil;

	@PostConstruct
	private void init() {
		geoCodingUtil = this;
		geoCodingUtil.properties = this.properties;
		geoCodingUtil.restTemplate = this.restTemplate;
	}

	// 调用示例
	public void getProperties(){
		System.out.println(geoCodingUtil.properties.getUrl());
	}

}
```

* 注解说明

	- `@PostConstruct`

			被`@PostConstruct`修饰的方法会在服务器加载`Servlet`的时候运行,并且只会被服务器调用一次，类似于`Servlet`的`init()`方法。被`@PostContruct`修饰的方法会在构造函数之后,`init()`方法之前运行
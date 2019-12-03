---
title: SpringBoot 使用自定义 Swagger2
toc: true
layout: SpringBoot
categories:
  - SpringBoot
tags:
  - SpringBoot
  - Swagger2
date: 2018-07-03 10:30
---
{{title}}
<!-- more -->
> <h3>SpringBoot 整合 Swagger2	| 自定义 swagger-spring-boot-starter</h3>

*  注: 传统的后端开发人员与其他前端或APP端需共同制定API接口文档，Swagger2 将文档变成可更新的在线版本，并且支持在线测试可以提高沟通效率和规范接口说明.

- 自定义 `swagger-spring-boot-starter` 的依赖，我所知道的分别有	

<a href="http://blog.battcn.com/2018/05/16/springboot/v2-config-swagger/" target="_blank">唐亚峰</a>

```xml
	<dependency>
	    <groupId>com.battcn</groupId>
	    <artifactId>swagger-spring-boot-starter</artifactId>
	    <version>1.4.5-RELEASE</version>
	</dependency>
```
<a href="https://github.com/SpringForAll/spring-boot-starter-swagger" target="_blank">Spring4all社区版</a>		

```xml
	<dependency>
		<groupId>com.spring4all</groupId>
		<artifactId>swagger-spring-boot-starter</artifactId>
		<version>1.7.0.RELEASE</version>
	</dependency>
```

**前提：** 创建 SpringBoot 项目(也可以是SpringMVC项目，配置方法另行百度咯)	

---

# 第一种：使用 `Swagger` 原生依赖配置		

## 创建`SpringBoot`应用，添加`pom.xml`依赖 	

项目创建完成完整依赖如下：	

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter</artifactId>
</dependency>

<dependency>
	<groupId>io.springfox</groupId>
	<artifactId>springfox-swagger2</artifactId>
	<version>2.7.0</version>
</dependency>

<dependency>
	<groupId>io.springfox</groupId>
	<artifactId>springfox-swagger-ui</artifactId>
	<version>2.7.0</version>
</dependency>

<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-test</artifactId>
	<scope>test</scope>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

## 创建 `Swagger2`配置类		

> 创建`API`组

- 注: 本文创建的是API组，故Swagger2Config配置类中有多个。只需要创建单个API组时无需创建两个 Docke @Bean实体类

```java
package com.learning.config;

import org.springframework.context.annotation.Bean;
import org.springframework.web.context.request.async.DeferredResult;

import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

 /**
	*  Swagger2 配置类,
	*  不使用@Configuration 注解是可以在 SwaggerApplication.class 中使用@Import注解代替
	*  @ClassName: Swagger2Config
	*  @Description: TODO
	*  @Created by time on 2018/07/19
	*/
	@Configuration
	public class Swagger2Config {
	
	/**
	 * 第一组 API
	 */
	private final String BASEPACKAGE ="com.learning.controller";

	private final String BASEPACKAGE2 ="com.learning.group";
	
	/**
	 * 定义单个 API
	 * 
	 */
	@Bean
	public Docket createApi() {
		
		return new Docket(DocumentationType.SWAGGER_2)
				.groupName("用户相关API")
				.genericModelSubstitutes(DeferredResult.class)
				.useDefaultResponseMessages(false)
				.forCodeGeneration(true)
				.apiInfo(apiInfo())
				.select()
				.apis(RequestHandlerSelectors.basePackage(BASEPACKAGE))
				.paths(PathSelectors.any())
				.build();
	}
	
	private ApiInfo apiInfo() {
		return new ApiInfoBuilder()
				.title("SpringBoot 官方版 Swagger 构建RESTful文档")
				.description("SpringBoot应用在线调试文档")
				.termsOfServiceUrl("http://www.baidu.com")
				.contact(new Contact("author", "http://baidu.com", ""))//名字，地址，邮箱
				.version("1.0")
				.build();
	}

	/**
	 * 定义 API 组
	 */
	@Bean
	public Docket innerApi() {
		return new Docket(DocumentationType.SWAGGER_12)
				.groupName("innerApi")
				.genericModelSubstitutes(DeferredResult.class)
				.useDefaultResponseMessages(false)
				.forCodeGeneration(true)
				.select()
				.apis(RequestHandlerSelectors.basePackage(BASEPACKAGE2))
				.paths(PathSelectors.any())
				.build()
				.apiInfo(innerApiInfo());
	}
	
	private ApiInfo innerApiInfo() {
		return new ApiInfoBuilder()
				.title("SpringBoot 官方版 Swagger 构建RESTful文档")
				.description("SpringBoot应用在线调试文档")
				.termsOfServiceUrl("http://www.baidu.com")
				.contact(new Contact("程序员DD", "http://blog.didispace.com", ""))
				.version("1.0")
				.build();
	}
}
```

> 修改`Application.class`中增加`@Import`		

```java
// 增加 Class 类注解
@Import(value= {Swagger2Config.class})
```

## 创建带有`Swagger`注解实体类	

```java
package com.learning.entity.vo;

import java.io.Serializable;

import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;

/**
  *  用户视图实体类
  *  @ClassName: UserVO
  *  @Description: TODO
  *  @Created by time on 2018/07/18
  */
@ApiModel(value = "用户实体类")
public class UserVO implements Serializable {

	private static final long serialVersionUID = 1L;

	@ApiModelProperty(value = "账户", required = true)
	private String account;

	@ApiModelProperty(value = "姓名", required = true)
	private String name;

	@ApiModelProperty(value = "昵称", required = true)
	private String nickName;

	public UserVO() {
	}

	public UserVO(String account, String name, String nickName) {
		this.name = name;
		this.nickName = nickName;
	}

	@Override
	public boolean equals(Object obj) {
		if (this == obj)
			return true;
		if (obj == null)
			return false;
		if (getClass() != obj.getClass())
			return false;
		UserVO other = (UserVO) obj;
		if (account == null) {
			if (other.account != null)
				return false;
		} else if (!account.equals(other.account))
			return false;
		if (name == null) {
			if (other.name != null)
				return false;
		} else if (!name.equals(other.name))
			return false;
		if (nickName == null) {
			if (other.nickName != null)
				return false;
		} else if (!nickName.equals(other.nickName))
			return false;
		return true;
	}

	public String getAccount() {
		return account;
	}

	public void setAccount(String account) {
		this.account = account;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public String getNickName() {
		return nickName;
	}

	public void setNickName(String nickName) {
		this.nickName = nickName;
	}

	@Override
	public String toString() {
		return "UserVO [account=" + account + ", name=" + name + ", nickName=" + nickName + "]";
	}

}
```

## 创建带有`Swagger`注解的`controller`		

**另一个`Controller`与下面的一致，就不贴代码了**

```java
package com.learning.controller;

import java.util.ArrayList;
import java.util.List;

import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import com.learning.entity.vo.UserVO;

import io.swagger.annotations.Api;
import io.swagger.annotations.ApiImplicitParam;
import io.swagger.annotations.ApiImplicitParams;
import io.swagger.annotations.ApiOperation;
import io.swagger.annotations.ApiResponse;
import io.swagger.annotations.ApiResponses;

/**
 * 
 * @ClassName: UserController
 * @Description: 用户管理
 * @author time
 * @date 2018/11/20
 */
@RestController
@RequestMapping(value = "/user")
@Api(value = "UserManager", tags = { "用户管理" })
public class UserController {

	/**
	 * 
	 * @Title: getUser
	 * @Description: 获取用户list
	 * @param userName
	 * @param password
	 * @return
	 * List<UserVO>
	 */
	@ApiOperation("获取用户相关信息")
	@ApiImplicitParams({
			@ApiImplicitParam(paramType = "query", name = "userName", dataType = "String", required = true, value = "用户的姓名", defaultValue = "张飞"),
			@ApiImplicitParam(paramType = "query", name = "password", dataType = "String", required = true, value = "用户的密码", defaultValue = "wangga") })
	@ApiResponses({ @ApiResponse(code = 400, message = "请求参数错误"),
			@ApiResponse(code = 404, message = "请求路径没有或者页面跳转路径错误") })
	@RequestMapping(value = "/getUser", method = RequestMethod.POST)
	public List<UserVO> getUser(@RequestParam("userName") String userName, @RequestParam("password") String password) {

		System.out.print("logger.in:getUser---------");

		UserVO info = new UserVO("test01", "测试账户一", "士兵");
		UserVO info2 = new UserVO("test02", "测试账户二", "将领");
		UserVO info3 = new UserVO("test03", "测试账户三", "元帅");

		List<UserVO> list = new ArrayList<>();
		list.add(info);
		list.add(info2);
		list.add(info3);

		return list;
	}

	/**
	 * 
	 * @Title: queryUserInfo
	 * @Description: 查询用户详细信息
	 * @param id
	 * @return
	 * Object
	 */
	@ApiOperation(value = "查询用户详细信息")
	@ApiImplicitParam(paramType = "path", name = "id", dataType = "long", required = true, value = "用户id", defaultValue = "1")
	@ApiResponses({ @ApiResponse(code = 400, message = "请求参数错误"),
			@ApiResponse(code = 404, message = "请求路径没有或者页面跳转路径错误") })
	@RequestMapping(value = "queryUserInfo/{id}", method = RequestMethod.GET)
	public Object queryUserInfo(@PathVariable("id") Long id) {
		UserVO info = new UserVO("士兵" + id, "00" + id, "张三" + id);
		return info;
	}

	/**
	 * 
	 * @Title: modifyUserInfo
	 * @Description: 修改用户信息
	 * @param id
	 * @param UserVO
	 * @return
	 * Object
	 */
	@ApiOperation(value = "修改用户信息")
	@ApiImplicitParams({
			@ApiImplicitParam(paramType = "query", name = "id", dataType = "long", required = true, value = "用户id", defaultValue = "1"),
			@ApiImplicitParam(paramType = "body", name = "User", dataType = "User", value = "修改的用户信息") })
	@ApiResponses({ @ApiResponse(code = 400, message = "请求参数错误"),
			@ApiResponse(code = 404, message = "请求路径没有或者跳转页面错误") })
	@RequestMapping(value = "modifyUserInfo", method = RequestMethod.POST)
	public Object modifyUserInfo(@RequestParam("id") Long id, @RequestBody UserVO UserVO) {
		System.out.print("-----------modifyUserInfo:" + UserVO.toString());
		UserVO.setAccount(++id + "");
		return UserVO;
	}

	/**
	 * 
	 * @Title: delUserById
	 * @Description: 删除用户信息
	 * @param id
	 * @return
	 * String
	 */
	@ApiOperation(value = "删除用户信息")
	@ApiImplicitParam(paramType = "path", name = "id", dataType = "long", required = true, value = "用户id", defaultValue = "1")
	@ApiResponses({ @ApiResponse(code = 400, message = "请求参数错误"),
			@ApiResponse(code = 404, message = "请求路径异常,或者跳转页面错误") })
	@RequestMapping(value = "delUserById/{id}", method = RequestMethod.DELETE)
	public String delUserById(@PathVariable(value = "id", required = true) Long id) {
		System.err.print("-----del");
		return "success";
	}

}
```

- `RESTful`接口

> ### `Swagger` 注解说明

* `@Api`： 描述Controller 	
	`value`：文档页面不显示
	`tags`:数组类型（{"...","..."}）；显示


* `@ApiIgnore`： 忽略该Controller，指不对当前类做扫描
* `@ApiOperation`： 描述Controller类中的method接口
* `@ApiParam`： 单个参数描述，与@ApiImplicitParam不同的是，他是写在参数左侧的。如（`@ApiParam(name = "username",value = "用户名") String username）`
* `@ApiModel`： 描述POJO对象
* `@ApiProperty`： 描述POJO对象中的属性值
* `@ApiImplicitParam`： 描述单个入参信息
* `@ApiImplicitParams`： 描述多个入参信息
* `@ApiResponse`： 描述单个出参信息
* `@ApiResponses`： 描述多个出参信息
* `@ApiError`： 接口错误所返回的信息

---

# 第二种: 使用自定义 `Swagger` 依赖配置		

## 创建`SpringBoot`项目	

<hr />

## 添加`pom.xml`中的依赖		

* 注: 替换上述的`swagger dependency`为下方依赖即可** 	

```xml
<dependency>
    <groupId>com.battcn</groupId>
    <artifactId>swagger-spring-boot-starter</artifactId>
    <version>1.4.5-RELEASE</version>
</dependency>
```

> 修改`application.yml`配置文件，定义在线接口显示内容	

```yaml
spring:
  swagger:
    base-package: com.learning.controller
    enabled: true
    title: Swagger API文档说明
    description: 在线调试文档
    version: 1.0
    contact:
      name: Time Machine
```

## 修改主函数`Application.class`，添加`@EnableSwagger2Doc`注解    

* 注: 只需要按照`Swagger`的注解创建`controller`与`entity`即可，无需`SwaggerConfig`配置类
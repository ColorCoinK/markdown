---
title: SpringBoot 整合在线API(Swagger2)
toc: true
layout: SpringBoot
categories:
  - SpringBoot
  - Swagger2
tags:
  - SpringBoot
  - Swagger2
date: 2019-02-28 11:00:13
---
{{title}}
<!-- more -->
# SpringBoot 整合 Swagger2 

## 创建SpringBoot 项目  

> 第一步：创建SpringBoot项目，启动应用看是否能够启动	

> 第二步：修改 pom.xml 为项目增加Swagger2支持   

```xml
<dependency>
	<groupId>io.springfox</groupId>
	<artifactId>springfox-swagger2</artifactId>
	<version>2.7.0</version>
</dependency>
<!-- https://mvnrepository.com/artifact/io.springfox/springfox-swagger-ui -->
<dependency>
	<groupId>io.springfox</groupId>
	<artifactId>springfox-swagger-ui</artifactId>
	<version>2.7.0</version>
</dependency>	
```

## 配置类

> 第三步：配置 Swagger2 配置类 	

```java
@Configuration
@EnableSwagger2
public class Swagger2 extends WebMvcConfigurerAdapter {

//配置需要扫描的注解包路径
private String basePackage="com.swagger.learing.controller";

@Bean
public Docket creatRestApi(){
return new Docket(DocumentationType.SWAGGER_2)
.apiInfo(apiInfo())
.select()
.apis(RequestHandlerSelectors.basePackage(this.basePackage))
.paths(PathSelectors.any())
.build();
}

/**
* 文档创建描述
* @return
*/
private ApiInfo apiInfo(){
return new ApiInfoBuilder()
			.title("API文档标题")
			.description("xx相关API")
			.termsOfServiceUrl("http://127.0.0.1:xx/swagger-ui.html";)
			.contact(new  Contact("name","url";,"email"))
				.version("版本号")
			.build();
	}
}	
```

**注解说明：**	

- `@Configuration`	声明类为配置类	

- `@EnableSwagger2`	用于开启Swagger注解	

## 设置 Controller 层注解

> 第四步: 为UserVo(返回结果实体)类添加 Swagger2 注解	

```java
package com.swagger.learing.domain;

import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;

@ApiModel(value = "用户模型实体类")
public class UserVo {

@ApiModelProperty(value = "账户", required = true)
private String account;
@ApiModelProperty(value = "姓名", required = true)
private String name;
@ApiModelProperty(value = "昵称", required = true)
private String nickName;

public UserVo() {
}

public UserVo(String account, String name, String nickName) {
	this.account = account;
	this.name = name;
	this.nickName = nickName;
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
	return "UserVo{" +
	"account='" + account + '\'' +
	", name='" + name + '\'' +
	", nickName='" + nickName + '\'' +
	'}';
	}
}
```

> 第五步：为Controller添加 Swagger2 生成在线API文档注解 
	
```java
package com.swagger.learing.controller;

import com.alibaba.fastjson.JSON;
import com.swagger.learing.domain.UserVo;
import io.swagger.annotations.*;
import org.springframework.web.bind.annotation.*;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

import java.util.ArrayList;
import java.util.List;

@RestController
@RequestMapping("user")
@Api("userController相关Api")
@EnableSwagger2 // 设置可被swagger识别显示
public class UserController {

   /**
	* 获取用户list
	* @param userName
	* @param password
	* @return
	*/
	@ApiOperation("获取用户相关信息")
	@ApiImplicitParams({
			@ApiImplicitParam(paramType = "query", name = "userName", dataType = "String", required = true, value = "用户的姓名", defaultValue = "张飞"),
			@ApiImplicitParam(paramType = "query", name = "password", dataType = "String", required = true, value = "用户的密码", defaultValue = "wangga") })
	@ApiResponses({ @ApiResponse(code = 400, message = "请求参数错误"),
			@ApiResponse(code = 404, message = "请求路径没有或者页面跳转路径错误") })
	@RequestMapping(value = "/getUser", method = RequestMethod.POST)
	public List<UserVo> getUser(@RequestParam("userName") String userName, @RequestParam("password") String password) {
		System.out.print("logger.in:getUser---------");
		UserVo info = new UserVo("test01", "测试账户一", "士兵");
		UserVo info2 = new UserVo("test02", "测试账户二", "将领");
		UserVo info3 = new UserVo("test03", "测试账户三", "元帅");

		List<UserVo> list = new ArrayList<>();
		list.add(info);
		list.add(info2);
		list.add(info3);

		return list;
	}

   /**
	* 查询用户详细信息
	* @param id
	* @return
	*/
	@ApiOperation(value = "查询用户详细信息")
	@ApiImplicitParam(paramType = "path", name = "id", dataType = "long", required = true, value = "用户id", defaultValue = "1")
	@ApiResponses({ @ApiResponse(code = 400, message = "请求参数错误"),
			@ApiResponse(code = 404, message = "请求路径没有或者页面跳转路径错误") })
	@RequestMapping(value = "queryUserInfo/{id}", method = RequestMethod.POST)
	public UserVo queryUserInfo(@PathVariable("id") Long id) {
		UserVo info = new UserVo("test" + id, "00" + id, "鼠标" + id);
		return info;
	}
  
   /**
	* 修改用户信息
	* @param id
	* @param userVo
	* @return
	*/
	@ApiOperation(value = "修改用户信息")
	@ApiImplicitParams({
			@ApiImplicitParam(paramType = "query", name = "id", dataType = "long", required = true, value = "用户id", defaultValue = "1"),
			@ApiImplicitParam(paramType = "body", name = "User", dataType = "User", value = "修改的用户信息") })
	@ApiResponses({ @ApiResponse(code = 400, message = "请求参数错误"),
			@ApiResponse(code = 404, message = "请求路径没有或者跳转页面错误") })
	@RequestMapping(value = "modifyUserInfo", method = RequestMethod.POST)
	public String modifyUserInfo(@RequestParam("id") Long id, @RequestBody UserVo userVo) {
		System.out.print("-----------modifyUserInfo:" + userVo.toString());
		userVo.setAccount(++id + "");
		return JSON.toJSONString(userVo);
	}

   /**
	* 删除用户信息
	* @param id
	* @return
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

按上述步骤配置，项目启动后访问地址：http://localhost:8080/swagger-ui.html

页面访问效果如下：
---
title: 项目集成 FineBI 应用系统 
toc: true
layout: blog
categories:
  - blog
  - Web Plugins
tags:
  - FineBI
date: 2019-02-27 15:20:13
---
FineBI后台Java集成报表页面
<!-- more -->
# 项目集成 FineBI 应用系统

* 概述  
  1. 安装 `FineBI` 客户端,或已有 `FineBI` 服务端;
  2. 使用`FineBI Web 页面集成`;

> 前提	

1. 安装`FIneBi` 找到`..\FineBI5.0\webapps\webroot` 文件夹，复制文件至`..\apache-tomcat-9.0.8\webapps\`目录下;

2. 复制`JDK` 1.8 或以上环境中的 `D:\Program Files\Java\jdk1.8.0_161\lib\tools.jar`文件到 `D:\Environment\Tomcat\apache-tomcat-9.0.8\lib` 文件夹下;

3. 在`dos`窗口执行`catalina.bat run`命令启动`tomcat` 或执行`start.sh`;

4. 访问`http://127.0.0.1:8080/webroot/decision` 出现`FineBI`登录配置页面.

* 如有变动或启动失败,可以查看官网地址<a href="http://help.finebi.com/doc-view-45" target="_blank">Tomcat 服务器部署</a>

## web 页面集成

> 实现思路

1. 配置登录`FineBI`系统信息,创建配置文件或设置配置文件到主配置文件中;
2. 创建登录跳转链接 >> 实现通过访问项目地址 >> 返回登录FineBI登录地址,携带登录信息 >> 重定向至`FineBI`系统
3. 完成集成

### 配置文件

> `finebi.properties`配置文件内容

```properties
finebi.path= webroot/decision/login/cross/domain
finebi.redirect= webroot/decision

#-------localhost-------
#finebi.ip=xxxx
#finebi.port=xxxx

#--------online---------
finebi.ip=xxxx
finebi.port=xxxxx
```

* `finebi.path`: 登录`FineBI`接口`URL`
* `finebi.redirect`: 登录成功后重定向`URL`
* `finebi.ip`: `FineBI` 服务部署地址
* `finebi.port`: `FineBI` 服务部署端口

> 实体类获取配置文件内容

```java
import java.io.BufferedInputStream;
import java.io.InputStream;
import java.util.Properties;

/**
 * 
 * @ClassName: FineBIProperties
 * @Description: FineBI 系统相关配置
 * @date 2018/12/05
 */
public class FineBIProperties {

	private static String ip;

	private static String port;

	private static String path;

	private static String redirect;

	static {
    // 读取配置文件
		Properties property = new Properties();
		try {
			ClassLoader cl = Thread.currentThread().getContextClassLoader();
			InputStream inputStream = new BufferedInputStream(cl.getResourceAsStream("finebi.properties"));
			property.load(inputStream);
			inputStream.close();
		} catch (Exception e) {
			e.printStackTrace();
		}
		ip = property.getProperty("finebi.ip");
		port = property.getProperty("finebi.port");
		path = property.getProperty("finebi.path");
		redirect = property.getProperty("finebi.redirect");
	}

  /**
   * 拼接登录FineBI 系统的登录URL
   **/
	public String loginFineBI() {
		String finbiURL = "http://" + ip + ":" + port + "/" + path + "?";
		return finbiURL;
	}

  /**
   * 拼接登陆后重定向的URL
   **/
	public String redirectURL() {
		String finbiURL = "http://" + ip + ":" + port + "/" + redirect;
		return finbiURL;
	}
}
```

### 访问API返回登录FinBI系统登录URL

```java
import java.util.HashMap;
import java.util.Map;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpSession;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

import com.sanss.finebi.FineBIProperties;
import com.sanss.user.entity.User;

/**
 * 
 * @ClassName: FineBIController
 * @Description: 接入 FineBI 相关
 * @date 2018/12/05
 */
@Controller
@RequestMapping(value = "finebi")
public class FineBIController {

	/**
	 * @return Object
	 * @Title: getLoginUrl
	 * @Description: 获取登录 FineBI 系统的路径,Path
	 */
	@ResponseBody
	@RequestMapping(value = "getLoginPath")
	public Object getLoginUrl(HttpServletRequest request) {
		Map<String, Object> result = new HashMap<String, Object>();

		FineBIProperties properties = new FineBIProperties();

    // 获取登录用户信息
		User user = new User("张三","1234");
		String userName = user.getUser_name();
		String password = user.getPasswd();

		// 获取登录 FineBI 的地址
		String url = properties.loginFineBI();

		// 拼接 FineBI 登录 url
		StringBuffer str = new StringBuffer(url);
		str.append(
				"fine_username=" + userName + "&fine_password=" + password + "&validity=-1"
						+ "&&callback=loginFineBI");

		result.put("url", str);
		result.put("redirect", properties.redirectURL());
		return result;
	}

}
```

* 登录的用户信息获取方式有可能不一样,需要根据自身现状进行调整。


### FineBI跳转页面

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>FineBI 页面</title>
<script type="text/javascript" src="jquery/jquery.min.js"></script>
<style type="text/css">
html {
	overflow: hidden;
}

html, body, iframe {
	width: 100%;
	height: 100%;
	margin: 0px;
	padding: 0px;
}

iframe {
	border: none;
}
</style>
</head>
<body>
	<iframe scrolling="no"></iframe>
	<script type="text/javascript">
    /** 获取项目名称 */
    function getProjectName(){
      var pathName = window.document.location.pathname;
      var projectName = pathName.substring(0,pathName.substr(1).indexOf('/')+1);
      return projectName;
    }

		//../js/util.js中定义的获取项目名称的方法
		var projectName = getProjectName();
		$(function() {
			$.getJSON(projectName + '/finebi/getLoginPath.do', function(data) {
				$.ajax({
					type : 'GET',
					url : data.url,
					dataType : 'jsonp',
					success : function(token) {
						loginFineBI(data, token);
					}
				});
			});
		})

		var loginFineBI = function(data, token) {
			$('iframe').attr("src", data.redirect);
		}
	</script>
</body>
</html>
```

### 其他的集成方式  

**注：**以上方式实现登录需要同步`项目连接数据库用户登录相关表`与`FineBI系统的用户信息表`才能实现登录,也可以采用`Token`或其他方式实现登录,有关其他的详细步骤可以查看官网<a href="http://help.finebi.com/doc-view-372.html" target="_blank">Web页面集成简单例子</a>
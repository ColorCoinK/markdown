---
title: SpringBoot配置https(xx.Jar、xx.war)
toc: true
layout: SpringBoot
categories:
  - SpringBoot
tags:
  - SpringBoot
  - Https
date: 2018/07/19 9:55:23
---
{{title}}
<!-- more -->
# SpringBoot配置https 与 War包配置https 

参考地址：[https://blog.csdn.net/u012702547/article/details/53790722][reference]

## SpringBoot 项目配置 *.jar 	

> 第一步：生成加密证书 	

- 方法：使用Java自带的命令,在cmd中执行以下语句  

`keytool -genkey -alias tomcat -storetype PKCS12 -keyalg RSA -keysize 2048 -keystore keystore.p12 -validity 3650`

> 参数说明	

1. -storetype	指定秘钥仓库类型	
2. -keyalg		指定证书生成算法签名，RSA是一种非对称加密算法 	
3. -keysize 	证书大小 	
4. -keystore	生成证书文件的存储路径 	
5. -validity	证书有效期 	

> 第二步：修改项目配置 

- 执行上述命令后，可在当前系统登录用户根目录下找到`keystore.p12`文件。<u>*将文件复制到springboot项目根目录下.*</u>  
同时需要对application.yml文件作出修改，增加如下内容。(也可使用xx.properties文件，文本内容修改为properties文件格式即可) 

> 配置文件修改(.yml格式文件，.properties文件修改只是使用了不同的书写方式如有不懂请自行百度) 

```yaml
server:
	ssl:
		key-store: keystore.p12
		key-store-password: 111111
		key-store-type: PKCS12
		key-alias: tomcat 
```

---

## Tomcat 部署 SpringBoot *.war 文件

> 第一步：生成加密证书
  	
- 方法：使用Java自带的命令,在cmd中执行以下语句  

`keytool -genkey -alias tomcat -storetype PKCS12 -keyalg RSA -keysize 2048 -keystore keystore.p12 -validity 3650`

> 第二步：  

1. 注释 8080 (tomcat默认端口为8080,如果修改为其他端口则注释修改后的端口)  

`<!-- <Connector port="7090" protocol="HTTP/1.1" connectionTimeout="20000"   redirectPort="8443" /> -->`

2. 取消注释 8433 端口配置,并修改为 443 端口(访问可不加端口设置)，修改tomcat中  
`D:\Environment\Tomcat\apache-tomcat-8.0.50\conf\server.xml` 需做如下配置。(tomcat路径为发布war文件的server.xml)

`keystoreFile="D:\Environment\Tomcat\apache-tomcat-8.5.31\conf\keystore.p12"     keystorePass="111111"`

* 注：修改后如下

```xml
<Connector port="443" protocol="org.apache.coyote.http11.Http11NioProtocol" maxThreads="150" SSLEnabled="true"
  keystoreFile="D:\Environment\Tomcat\apache-tomcat-8.5.31\conf\keystore.p12" keystorePass="111111" >   
      <!--
        <SSLHostConfig>
            <Certificate certificateKeystoreFile="conf/localhost-rsa.jks" type="RSA" />
        </SSLHostConfig>
      -->
</Connector>
  <!-- 修改8443端口为443 -->
  <Connector port="8009" protocol="AJP/1.3" redirectPort="443" />
```

[reference]: https://blog.csdn.net/u012702547/article/details/53790722
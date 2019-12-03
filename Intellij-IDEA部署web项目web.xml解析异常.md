---
title: Intellij IDEA 部署 Web 项目时,web.xml 无法正常解析导致 jsp 显示错误	
toc: false
layout: blog
categories:
  - blog
tags:
  - Intellij IDEA
date: 2019-02-28 11:00:13
---
{{title}}
<!-- more -->
# Intellij IDEA 部署 Web 项目时,web.xml 无法正常解析导致 jsp 显示错误	

> 背景	

1. intellij IDEA 将应用打成`war`包可以正常运行及显示 

2. eclipse 使用同样的`tomcat | JDK`运行正常 

3. intellij IDEA 使用`tomcat`运行失败 

> 错误信息	

Intellij IDEA 控制台信息 

```html
 java.io.FileNotFoundException: Could not resolve XML resource [null] with public ID [-//Sun Microsystems, Inc.//DTD JSP Tag Library 1.2//EN], system ID [http://java.sun.com/dtd/web-jsptaglibrary_1_2.dtd] and base URI [jar:file:/D:/Code/IDE-workspace/iptvView/target/WEB-INF/lib/standard-1.1.2.jar!/META-INF/c-1_0-rt.tld] to a known, local entity.
	at org.apache.tomcat.util.descriptor.LocalResolver.resolveEntity(LocalResolver.java:155)
```

Chrome 浏览器页面信息		

```html
HTTP Status 500 - The absolute uri: http://java.sun.com/jsp/jstl/core cannot be resolved in either web.xml or the jar files deployed with this application
type Exception report

message The absolute uri: http://java.sun.com/jsp/jstl/core cannot be resolved in either web.xml or the jar files deployed with this application

description The server encountered an internal error that prevented it from fulfilling this request.

exception

org.apache.jasper.JasperException: The absolute uri: http://java.sun.com/jsp/jstl/core cannot be resolved in either web.xml or the jar files deployed with this application
	org.apache.jasper.compiler.DefaultErrorHandler.jspError(DefaultErrorHandler.java:55)
	org.apache.jasper.compiler.ErrorDispatcher.dispatch(ErrorDispatcher.java:277)
	org.apache.jasper.compiler.ErrorDispatcher.jspError(ErrorDispatcher.java:75)
	org.apache.jasper.compiler.TagLibraryInfoImpl.generateTldResourcePath(TagLibraryInfoImpl.java:250)
	org.apache.jasper.compiler.TagLibraryInfoImpl.<init>(TagLibraryInfoImpl.java:125)
	org.apache.jasper.compiler.Parser.parseTaglibDirective(Parser.java:421)
	org.apache.jasper.compiler.Parser.parseDirective(Parser.java:479)
	org.apache.jasper.compiler.Parser.parseElements(Parser.java:1435)
	org.apache.jasper.compiler.Parser.parse(Parser.java:139)
	org.apache.jasper.compiler.ParserController.doParse(ParserController.java:227)
	org.apache.jasper.compiler.ParserController.parse(ParserController.java:100)
	org.apache.jasper.compiler.Compiler.generateJava(Compiler.java:201)
	org.apache.jasper.compiler.Compiler.compile(Compiler.java:358)
	org.apache.jasper.compiler.Compiler.compile(Compiler.java:338)
	org.apache.jasper.compiler.Compiler.compile(Compiler.java:325)
	org.apache.jasper.JspCompilationContext.compile(JspCompilationContext.java:580)
	org.apache.jasper.servlet.JspServletWrapper.service(JspServletWrapper.java:363)
	org.apache.jasper.servlet.JspServlet.serviceJspFile(JspServlet.java:396)
	org.apache.jasper.servlet.JspServlet.service(JspServlet.java:340)
	javax.servlet.http.HttpServlet.service(HttpServlet.java:790)
	org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:52)
	org.springframework.web.servlet.view.InternalResourceView.renderMergedOutputModel(InternalResourceView.java:238)
	org.springframework.web.servlet.view.AbstractView.render(AbstractView.java:263)
	org.springframework.web.servlet.DispatcherServlet.render(DispatcherServlet.java:1208)
	org.springframework.web.servlet.DispatcherServlet.processDispatchResult(DispatcherServlet.java:992)
	org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:939)
	org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:856)
	org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:936)
	org.springframework.web.servlet.FrameworkServlet.doGet(FrameworkServlet.java:827)
	javax.servlet.http.HttpServlet.service(HttpServlet.java:687)
	org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:812)
	javax.servlet.http.HttpServlet.service(HttpServlet.java:790)
	org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:52)
note The full stack trace of the root cause is available in the Apache Tomcat/8.0.52 logs.

Apache Tomcat/8.0.52
```

> 项目 `xxx/WEB-INF/lib` 中不存在相关`jar`包	

可以从 <a href="https://mvnrepository.com/artifact/javax.servlet/jstl/1.2" target="_blank" >mavenrepository地址</a> 中找到相应`jar`包,添加相关依赖到`pom.xml`文件中并重新编译，查看问题是否解决。

**注：** 通过添加缺失`jar`可以解决的就不用往下看了

> 项目依赖中存在相关依赖,依然无法正常显示页面	

 >> 1. 找到项目配置的 `tomcat` 下 `xx\apache-tomcat-8.0.52\conf\context.xml`文件

 >> 2. 修改文件,添加如下内容 

 <Context <u> xmlBlockExternal="false" </u> >

**注：** 只需要添加下划线中的 `xmlBlockExternal="false"` 到 `Context` 标签即可

> 总结： 	

1. 出现`jsp`页面显示错误,一般都是缺少 `jar` 包所致。可检查在`tomcat`或编译后的文件夹中是否能够找到相关依赖。

2. 上述异常中缺失的`jar`有,然而本次并不是缺少`jar`的错误

```
standard-1.1.2.jar、jstl-1.2.jar、jersey-server-1.9.jar、jstl-impl.jar
```

3. 本博客无法解决疑问时,可简练相关关键字再搜索。

> 参考博客  <a href="https://blog.csdn.net/wb96a1007/article/details/71665683" target="_blank">新版Tomcat无法解析web.xml</a>
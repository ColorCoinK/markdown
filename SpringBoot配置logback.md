---
title: SpringBoot + logback 日志配置
toc: true
layout: SpringBoot
categories:
  - SpringBoot
tags:
  - SpringBoot
  - Logback
date: 2018/07/19 9:55:23
---
{{title}}
<!-- more -->
# `SpringBoot` 使用 `logback-spring.xml` 生成日志文件		

* 注: 

<a href="https://blog.csdn.net/inke88/article/details/75007649" target="_blank">参考配置1</a>
<a href="https://blog.battcn.com/2018/04/23/springboot/v2-config-logs" target="_blank">参考配置2</a>

## `logback`配置文件命名		

> 自定义日志配置		

* 创建名为 `logback.xml`、`logback-xxx.xml`的文件,需要在 `application.yml/application.properties`文件中指定加载的配置文件路径		

```yml
logging:
  config: classpath:logback.xml
```

* 注：官方推荐将`logback-spring.xml`放在根目录下,可以省略上述步骤. 

## `logback-spring.xml`配置 		

1. 在`application.yml`中指定日志文件输出路径、文件名		

```yml
logback: 
  logdir: D:/project/logs
  name: project
```

* 注: 当前环境为`window`为包含盘符的绝对路径,`linux`环境需要修改为不含盘符的绝对路径

2. `logback-spring.xml`配置文件内容	

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="tue" scanPeriod="60 seconds"
	debug="false">
	
	<!-- application.yml 传递参数，不能使用logback 自带的<property>标签 -->
	<!-- 日志输出路径,在yml文件中配置 -->
	<springProperty scope="context" name="logdir" source="logback.logdir"/>
	<!-- 文件名 -->
	<springProperty scope="context" name="logName" source="logback.name"/>
	
	<!-- 输出到控制台 Consoleappender -->
	<appender name="consoleLog" class="ch.qos.logback.core.ConsoleAppender">
		<!-- 展示格式 layout -->
		<layout class="ch.qos.logback.classic.PatternLayout">
			<pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
		</layout>
	</appender>

	<!-- infoLog 输出 -->
	<appender name="fileInfoLog" class="ch.qos.logback.core.rolling.RollingFileAppender">
		<!-- 如果只是想要 info 级别的日志,只是过滤 info 还是会输出 error 日志,因为 Error 的日志级别更高。
		 使用下面的策略,可以避免输出  Error 日志 -->
		<filter class="ch.qos.logback.classic.filter.LevelFilter">
			<!-- 过滤 Error -->
			<level>ERROR</level>
			<!-- 匹配到就禁止 -->
			<onMatch>DENY</onMatch>
			<!-- 没有匹配到就允许 -->
			<onMismatch>ACCEPT</onMismatch>
		</filter>
		
		<!-- 日志名称,如果没有 File 属性,name只会使用FileNamePattern的文件路径规则
			如果同时又<File>和<FileNamePattern>,那么当天日志时<File>.明天会自动把今天的日志名改为今天的日期
			即：<File> 的日志都是当天的。
		  -->
		<File>${logdir}/info.${logName}.log</File>
		<!-- 滚动策略,按照时间滚动 TimeBasedRollingPolicy -->
		<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
			<!-- 文件路径,定义了日志的切分方式 把每一天的日志归档到一个文件中,以防止日志填满整个磁盘空间 -->
			<FileNamePattern>${logdir}/info.${logName}.%d{yyyy-MM-dd}.log</FileNamePattern>
			<!-- 设置 日志保留时间 30天 -->
			<maxHistory>30</maxHistory>
			<!-- 用来指定日志文件的上限大小,到上限之后会删除旧日志 -->
			<!-- <totalSizeCap>1GB</totalSizeCap> -->
		</rollingPolicy>
		
		<!-- 日志输出编码格式化 -->
		<encoder>
			<charset>UTF-8</charset>
			<pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
		</encoder>
	</appender>
	
	<!-- Error 级别日志文件 -->
	<appender name="fileErrorLog" class="ch.qos.logback.core.rolling.RollingFileAppender">
		<!-- 如果只是想要 Error 级别的日志,那么需要过滤一下.默认是info 级别的,ThresholdFilter -->
		<filter class="ch.qos.logback.classic.filter.ThresholdFilter" >
			<level>Error</level>
		</filter>
		<!-- 日志名称,如果没有 File 属性,name只会使用FileNamePattern的文件路径规则
			如果同时又<File>和<FileNamePattern>,那么当天日志时<File>.明天会自动把今天的日志名改为今天的日期
			即：<File> 的日志都是当天的。
		  -->
		<File>${logdir}/error.${logName}.log</File>
		<!-- 滚动策略,按照时间滚动 TimeBasedRollingPolicy -->
		<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
			<!-- 文件路径,定义了日志的切分方式 把每一天的日志归档到一个文件中,以防止日志填满整个磁盘空间 -->
			<FileNamePattern>${logdir}/error.${logName}.%d{yyyy-MM-dd}.log</FileNamePattern>
			<!-- 设置 日志保留时间 30天 -->
			<maxHistory>30</maxHistory>
			<!-- 用来指定日志文件的上限大小,到上限之后会删除旧日志 -->
			<!-- <totalSizeCap>1GB</totalSizeCap> -->
		</rollingPolicy>
		
		<!-- 日志输出编码格式化 -->
		<encoder>
			<charset>UTF-8</charset>
			<pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
		</encoder>
	</appender>
	
	<!-- 开发环境日志级别为DEBUG ,可以使用逗号分隔列表指定多个配置文件 -->
	<springProfile name="dev">
		<root level="INFO">
			<appender-ref ref="consoleLog" />
			<appender-ref ref="fileInfoLog" />
			<appender-ref ref="fileErrorLog" />
		</root>
	</springProfile>
</configuration>
```

###  配置文件标签说明		

> 1. 根节点`<configuration>`包含的属性	

* scan:当此属性设置为true时，配置文件如果发生改变，将会被重新加载，默认值为true。
* scanPeriod:设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。当scan为true时，此属性生效。默认的时间间隔为1分钟。
* debug:当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false。		

- 注:  依照本文配置,可在指定路径中生成自定义的日志文件名(`info`与`error`日志分开,默认当前日志为`application.yml`中的`logging.name`作为文件名可自行定义)。历史日志文件名 增加日期格式 
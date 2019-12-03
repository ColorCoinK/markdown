---
title: SpringBoot 配置定时任务
toc: true
layout: SpringBoot
categories:
  - SpringBoot
tags:
  - SpringBoot
  - EnableScheduling
date: 2019-02-28 11:00:13
---
{{title}}
<!-- more -->
# SpringBoot 创建定时任务	

**要求**

	* 已创建SpringBoot项目
	* JDK 版本 1.8 及以上(非必须)
	* Maven 版本 3.2+


## 添加依赖	
	
```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter</artifactId>
</dependency>

<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-test</artifactId>
	<scope>test</scope>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-devtools</artifactId>
	<optional>true</optional>
</dependency>
```

* 注: 进行 `选中项目右键 > Maven > Update Project` 操作后如果 JDK 版本被修改，在`pom.xml`中添加

```xml
<java.version>1.8</java.version>

<build>
	<plugins>
		<plugin>
			<artifactId>maven-compiler-plugin</artifactId>
			<configuration>
				<source>${java.version}</source>
				<target>${java.version}</target>
			</configuration>
		</plugin>
		</plugin>
	</plugins>
</build>
```

## 启动类添加注解

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.scheduling.annotation.EnableScheduling;

//定时任务注解
@EnableScheduling
@SpringBootApplication
public class Application {

	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}
}
```

## 创建定时任务实现类	

1. 定时任务  

```java
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

@Component
public class SyncingTask {

	private int count = 0;

	@Scheduled(cron = "*/6 * * * * ?")
	public void process() {
		System.out.println("this is scheduler task runing" + (count++));
	}
}
```

2. 定时任务  

```java
import java.text.SimpleDateFormat;
import java.util.Date;

import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

@Component
public class PrintTask {

	private static final SimpleDateFormat sdf = new SimpleDateFormat("HH:mm:ss");

	@Scheduled(fixedRate = 6000)
	public void reportCurrentTime() {
		System.out.println("现在时间：" + sdf.format(new Date()));
	}

}
```

* 运行程序即可在控制台看到类似输出		

```java
2018-07-16 16:44:57.045  INFO 9352 --- [  restartedMain] s.a.ScheduledAnnotationBeanPostProcessor : No TaskScheduler/ScheduledExecutorService bean found for scheduled processing
现在时间：16:44:57
2018-07-16 16:44:57.045  INFO 9352 --- [  restartedMain] com.learning.Application  : Started Application in 0.49 seconds (JVM running for 0.931)
this is scheduler task runing0
现在时间：16:45:03
this is scheduler task runing1
现在时间：16:45:09
this is scheduler task runing2
现在时间：16:45:15
this is scheduler task runing3
现在时间：16:45:21
```

* 示例可参见官方地址	<a href="http://spring.io/guides/gs/scheduling-tasks/" target="_blank" >Scheduling Tasks</a>

## `@Scheduled`注解说明	

- `@Scheduled(fixedRate = 5000)`：上次开始执行时间点之后5秒再执行
- `@Scheduled(fixedDelay = 5000)`：上次执行完毕时间点之后5秒再执行
- `@Scheduled(initialDelay = 1000, fixedRate = 5000)`：第一次延迟1秒后执行，之后按`fixedRate`的规则每5秒执行一次
- `@Scheduled(cron = "*/6 * * * * *")`：通过<a href="https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/scheduling/support/CronSequenceGenerator.html" target="_blank"> cron </a> 表达式定义规则

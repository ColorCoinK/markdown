---
title: Pom 文件常用配置
toc: true
layout: blog
categories:
  - blog
  - Maven
tags: 
  - Maven
  - pom
date: 2019-03-04 11:03:00
---
{{title}}
<!-- more -->

# pom.xml 常用的配置值

## properties

```xml
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <java.version>1.8</java.version>
    <swagger.version>2.7.0</swagger.version>
</properties>
```

## plugin

```xml
  <build>
    <plugins>
      <plugin>
        <artifactId>maven-compiler-plugin</artifactId>
        <configuration>
          <source>${java.version}</source>
          <target>${java.version}</target>
        </configuration>
      </plugin>
    </plugins>
  </build>
```

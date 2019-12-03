---
title: impala 使用记录	
toc: true
layout: blog
categories:
  - blog
  - Java
tags:
  - Java
  - Memory Overflow
date: 2019-02-28 11:00:13
---
{{title}}
<!-- more -->
# 大数据量查询,使用 `ResultSet` 出现 `JVM` 内存溢出	

> 解决方案	

* 修改前代码	

```java
statement = connection.prepareStatement(sql);
```

修改后代码

```java
statement = connection.prepareStatement(sql, ResultSet.TYPE_FORWARD_ONLY, ResultSet.CONCUR_READ_ONLY);
```
---
title: impala 使用记录	
toc: true
layout: blog
categories:
  - blog
  - Impala
tags:
  - Impala
date: 2019-02-28 11:00:13
---
{{title}}
<!-- more -->
# impala 使用记录	

# 基本命令	

> 进入 `impala`

	impala-shell	

> 退出 `impala`	

	exit;

> 展示所有表		

	show tables;

> 描述表结构	

	desc table_name;

**注:**
1.  每个`SQL`语句结束时需要加 ';',当数据量大时可以使用`limit`;
2.  查询时尽量使用数据分区字段,可以有效减少查询时间时间
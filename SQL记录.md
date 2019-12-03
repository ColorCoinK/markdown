---
title: SQL 常用函数
toc: true
layout: layout
categories:
  - blog
  - SQL
tags:
  - SQL
date: 2019-03-04 11:18:10
---
{{title}}
<!-- more -->
# `SQL` 常用函数及错误记录  

> 获取`sysdate`前几个月时间 

```sql
 -- 时间发生在 2019/1/29,查询(ORA-01839:指定月份的日期无效)
 -- 关键在于其他之前时间查询都没问题,就1月不行.有待解答
 select to_char(sysdate - interval '11' month, 'yyyy/mm') from dual 
  
 /* 替代SQL */

 select to_char(sysdate - interval '11' month, 'yyyy/mm') from dual 
```
```
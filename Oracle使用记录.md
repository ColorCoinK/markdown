---
title:  Oracle 使用笔记
toc: true
layout: blog
categories:
  - blog
  - Oracle
tags:
  - Oracle
date: 2019-02-28 11:00:13
---
{{title}}
<!-- more -->

# `Oracle`使用笔记

## 触发器	

* 查询 `t_device`表是否有关联触发器,查 `all_triggers` 表得到 `trigger_name`	

	`select trigger_name from all_triggers where table_name = 'XX';`

* 根据 `trigger_name` 查询出触发器详细信息		

	`select text from all_source where type = 'TRIGGER' AND name = 'TR_XXX';`

## `SQL`查看`CLOB`类型内容	

`SQL : select dbms_lob.substr(t.boundary_point) from t_station_boundary t;`

* 说明 :`dbms_lob.substr`将大文本转换字符类型读出来
> 引用地址：<a href="https://www.cnblogs.com/Grand-Jon/p/7389427.html" target="_blank">Oracle的CLOB大数据字段类型</a>

## `SQL`按月份统计数据	

> 引用地址: <a href="https://www.cnblogs.com/ymj126/p/4501061.html" target="_blank">Oracle按月统计数据</a>

## `Oracle`时间转换为5分钟粒度	

```sql
select to_char(watchtime, 'yyyy/MM/dd hh24') || ':' ||
       lpad(floor(to_char(watchtime, 'mi') / 5) * 5, 2, 0) watchtime,
       num
  from tm_num_watch order by watchtime asc;
```
## `Oracle`将一个字段拆分为两个字段在同行显示结果	

```sql
-- 获取行政区中心点坐标
select station_name,
       regexp_substr(t.central_point, '[^,]+', 1, 1) lat,
       regexp_substr(t.central_point, '[^,]+', 1, 2) lng
  from t_station_boundary t;
```

![Oracle使用正则拆分字段值][oracle_regexp_substr]

[oracle_regexp_substr]: ../../../images/oracle-regexp_substr.png

## 时间计算查询

```sql
-- 相差月份查询
select ceil((months_between(sysdate,'26-7月-2014'))) months from dual

-- 天数查询
select ceil((to_date('2018/5/24','yyyy/mm/dd') - to_date('2014/7/26','yyyy/mm/dd'))) days from dual    

-- 小时查询
select ceil((to_date('2018/5/24', 'yyyy/mm/dd') - to_date('2014/7/26', 'yyyy/mm/dd'))*24) hoursfrom dual;

-- 相差时间查询
select *  from (
  (select ceil((months_between(sysdate, '26-7月-2014'))) as "相差月数" from dual)),
  (select ceil(sysdate - to_date('2014/7/26', 'yyyy/mm/dd')) as "相差天数" from dual),
  (select ceil((sysdate - to_date('2014/7/26', 'yyyy/mm/dd')) * 24) as "相差小时"  from dual);
```

> 引用 [Oracle 两个时间相减][oracle_date_difference]

[oracle_date_difference][https://blog.csdn.net/redarmy_chen/article/details/7351410]

**注：**
  1. Oracle 两个时间相减默认的是天数;
  2. 两个时间相减的差 * 24 是得到的是 小时(hours),依次类推得到的相应的时间差.

## 查询加上前缀

```sql
-- concat 函数
select concat('_',USER_AD) USER_AD, longitude, latitude, grid_x, grid_y from t_advertising;
```

## `Oracle`使用`merge into`

- 索引

```sql
-- 创建索引
create index meshing_user_ad_idx on T_MESHING(USER_AD);

-- 删除索引
drop index meshing_user_ad_idx;
```

- `merge into语法`

```sql
merge into t_meshing t1
using (select :name user_ad, :longitude longitude, :latitude latitude, :gridX gridX, :gridY gridY
       from dual) t2
on (t1.user_ad = t2.user_ad)
when matched then
    update
    set t1.longitude=t2.longitude,
        t1.latitude = t2.latitude,
        t1.grid_x   = t2.gridx,
        t1.grid_y   = t2.gridy
when not matched then
    insert (t1.user_ad, t1.longitude, t1.latitude, t1.grid_x, t1.grid_y)
    values (t2.user_ad, t2.longitude, t2.latitude, t2.gridx, t2.gridy);
```
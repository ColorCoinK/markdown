---
title: Java使用正则表达式仅获取中文
toc: true
layout: blog
categories:
  - Blog
  - Java
tags:
  - Blog
  - Java
  - Regex
date: 2019-03-08 09:16:47
---
Java 使用正则表达式获取字符串中的中文,将字母、数字、下划线等去除.字符串为`现网电视剧订购series_order`
<!-- more -->

# Java 代码

```java
public static void main(String[] args) {
  String jsonStr = "[{\"number\":16,\"name\":\"现网电视剧订购series_order\"},{\"number\":17,\"name\":\"电影详情页movie_detail\"},{\"number\":19,\"name\":\"限免专区biz_30127402\"},{\"number\":19,\"name\":\"综艺详情variety_detail\"},{\"number\":24,\"name\":\"电视剧记录与收藏series_fav\"},{\"number\":24,\"name\":\"热门推荐biz_79818028\"},{\"number\":38,\"name\":\"电影专区biz_17101799\"},{\"number\":49,\"name\":\"电视剧详情页series_detail\"},{\"number\":53,\"name\":\"现网电视剧播放series_play\"},{\"number\":103,\"name\":\"热门推荐biz_87847639\"}]";
  List<rank> result = JSONArray.parseArray(jsonStr, rank.class);
  System.out.println(result);
  String name = "";
  String regex = "[a-zA-Z]";
  String[] array = new String[3];
  for (rank rank : result) {
    name = rank.getname();
    array = name.split(regex);
    rank.setname(array[0]);
  }
  System.out.println(result.toString());
}
```
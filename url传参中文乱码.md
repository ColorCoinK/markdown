---
title: 页面传参中文参数乱码
toc: true
layout: blog
categories:
  - blog
  - HTML
tags:
  - Java
  - CharacterEncoding
date: 2018/07/19 9:55:23
---
{{title}}
<!-- more -->
# 页面传递中文参数乱码解决

> 方案一: 接收端设置编码格式	

1. 使用 `request.setCharacterEncoding('utf-8');`
2. 使用 `request.getParameter("handlerType").getBytes("ISO-8859-1"),"utf-8")` 接受传递的参数值


---
title: HTML log 
toc: true
layout: blog
categories:
  - blog
  - HTML
tags:
  - HTML
date: 2019-02-27 15:20:13
---
{{title}}
<!-- more -->
# HTML 应用记录	

## `css`获取奇偶行数		

* 奇数行：`td:nth-child(odd){}`
* 偶数行：`td:nth-child(even){}`

	**css选择器：**<a href="http://www.w3school.com.cn/cssref/css_selectors.asp" target="_blank">参考地址</a> 

## 跳转`html`页面	

```js
parent.window.location.href = "../main/index.do";
```

## `border`设置渐变色	

```css
.customborder {
	border-image: -webkit-linear-gradient(#F80, #2ED) 20 20;
	border-image: -moz-linear-gradient(#F80, #2ED) 20 20;
	border-image: -o-linear-gradient(#F80, #2ED) 20 20;
	border-image: linear-gradient(#F80, #2ED) 20 20;
}
```

## 数组首尾添加空对象

```js
  // 头部添加
	result.unshift({});
	// 数组 尾部添加
	result.push({});
```

## 将XML字符串转为JSON对象

* 下载插件`http://www.kawa.net/works/js/xml/objtree-e.html#download`
* 按照案例实现即可
---
title: Jquery 使用日志
toc: true
layout: blog
categories:
  - Blog
  - JQuery
tags:
  - JQuery
date: 2019-02-28 11:00:13
---
{{title}}
<!-- more -->
# Jquery 使用笔记

## 文件加载后进行初始化 

```js
$(function () {
	// 本月产品类型
	monthTypes();

	//本月产品订购次数排名
	monthRanked();

	//最近24小时订购次数
	h24Rank();

	//本月产品订购收入排名
	mpincomeRank();

	/**
	 * 定时刷新数据
	 */
	setInterval(function () {
		setTimeout(monthTypes(), Math.random() * 100);
		setTimeout(monthRanked(), Math.random() * 100);
		setTimeout(h24Rank(), Math.random() * 100);
		setTimeout(mpincomeRank(), Math.random() * 100);
	}, 1000 * 60 * 5);

})
```

## `list`遍历   

> `each`

```js
    $.each(result, function(index, item) {
        // result[index] = item
        console.log(index);
    });
```

> `map`

```js
    result.map(function(item){
        console.log(item)
    })
```

> `for`

## 数字滚动插件 

> `GitHub` 地址


> 参照博客  
<a herf="https://segmentfault.com/a/1190000010060162" target="_blank">数字滚动显示插件地址</a>

## 延迟加载`*.js`文件

```js
<script type="text/javascript">
  function downloadJSAtOnload() {
    var element = document.createElement("script");
    element.src = "*.js";//相对路径
    document.body.appendChild(element);
  }

  if (window.addEventListener) {
    window.addEventListener("load", downloadJSAtOnload, false);
  } else if (window.attachEvent) {
    window.attachEvent("onload", downloadJSAtOnload);
  } else {
    window.onload = downloadJSAtOnload;
  }
</script>
```
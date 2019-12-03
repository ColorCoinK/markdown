---
title: Echarts使用总结
toc: true
layout: blog
categories:
  - blog
  - Web Plugins
tags:
  - Echarts
date: 2019-02-27 15:20:13
---

{{title}}

<!-- more -->
# `Echarts`使用笔记	

## 设置默认选中显示数据	

```js
option ={
	legend:{
		selected : {
			'XX':false	//不想显示的设为false
		}
	}
}
```

## 设置`Y`轴数值 `1000`显示为`1K`		

```js
yAxis : {
	type : 'value',
	axisLabel : {
		color : '#fff',
		formatter : function(value, index) {
			var thousand;
			if (value >= 1000) {
				thousand = value / 1000 + 'k';
			} else {
				thousand = value
			}
			return thousand;
		}
	}
}
```

## 设置超长字符串为`...`	


## 设置`echarts 柱状图颜色`	

```js
	option:{
		color: ['#003366', '#006699', '#4cabce', '#ccc']
	}
```

## 图标点击事件监听

>  <a href="https://www.echartsjs.com/zh/api.html#events" target="_blank">文档地址</a>

```js
/* 使用文档相同的监听不知道为什么无效,需要改为如下写法 */
// 始呼接通率监听事件
myCharts.getZr().on('click', function () {
	console.info('监听点击事件')
});
```

## 坐标轴赋值

```js
function myCharts(params) {
	var url="";
  $.getJSON(url, params,l
      function (result) {
        op.setOption({
          xAxis: {
            data: result.map(function (item) {
              return item.time;
            })
          },
          series: [{
            data: result.map(function (item) {
              return item.eci_rate;
            })
          }, {
            data: result.map(function (item) {
              return item.ne_rate;
            })
          }]
        });
      });
}
```

## 饼图

```js
// 使用自定义的echarts theme
var failed_pie_charts = echarts.init(document.getElementById("failed_pie"),'macarons');

$.getJSON(url, params, function (result) {
    var failed_reason = [];

    $.each(result, function (index, items) {
      failed_reason.push({
        name: items.QUERY_TYPE,
        value: items.RATE
      })
    });

    failed_pie_charts.setOption({
      legend: {
        data: result.map(function (item) {
          return item.QUERY_TYPE
        })
      },
      series: {
        data: failed_reason
      }
    })
  })
```

---
title: Eclipse | STS(Spring Tool Suite)开发工具配置
toc: true
layout: blog
categories:
  - blog
tags:
  - Eclipse
date: 2019-02-27 15:20:13
---
{{title}}
<!-- more -->

# 开发环境中`Eclipse`、`STS(Spring Tool Suite)`通用配置

> `Java`实体类，重写`toString()`.生成`JSON`字符串配置	

1. `Eclipse` 快捷键 `Alt + S`  选择 `Generate toString`，重写`toString()`方法	

	![步骤一][choose]

2. 添加自定义的`format`格式，选择`Edit`

	![步骤二][custome]

	- 格式化文本内容

	```java
	{"${member.name()}":"${member.value}", "${otherMembers}"}
	```

3. 新的`template`，输入`Name`和`Pattern`

	![步骤三][input]

4. 点击`OK`，完成创建。

	![步骤四][create]

5. 回到第一个页面，默认选中刚刚创建的`JSONString`，点击`Generate`完成创建
   
	![步骤五][apply]
   
6. 创建完成后效果
   
	![结果][result]

[choose]: /images/step1.png
[custome]: /images/step2.png
[input]: /images/step3.png
[create]: /images/step4.png
[apply]: /images/step5.png
[result]: /images/step6.png
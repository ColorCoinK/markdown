---
title: Shell log
toc: true
layout: blog
categories:
  - blog
  - Shell
tags:
  - Shell
date: 2019-02-28 11:00:13
---
{{title}}
<!-- more -->
```sh
#!/bin/bash
you_name="runoob.com";
echo "Hello : "${you_name};

# 只读变量的值不能被改变
myUrl="http://www.baidu.com";
readonly myUrl;
# myUrl="http://runoob.com";

# 删除变量

unset you_name;
echo ${you_name};

# 获取字符串长度
string="abcd";
echo ${#string};

# 查找子字符串
string="runoob is a great site";
echo `expr index "${string}" sg`;

# 定义数组
track=("min" "mid" "max");
track[5]="io流";
valuen=${track[5]};
echo ${valuen};
echo ${#track[@]};
```
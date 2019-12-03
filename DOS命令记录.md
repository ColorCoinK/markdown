---
title: Doc 常用命令记录
toc: true
layout: blog
categories:
  - Blog
  - Doc
tags:
  - Doc
date: 2019-02-27 15:20:13
---
{{title}}
<!-- more -->

# DOC 命令 log 

## 删除文件夹及文件夹下所有文件和文件夹		

```dos
rd /s /q folder
```

## 删除指定后缀的文件	

```dos
del /a /f /q file path\*.xx
```

## 查看当前文件加下所有文件夹及文件列表		

```dos
dir
```

## 清空 `DOS` 界面	

```dos
cls
```

## 文件夹	

* 进入某个文件夹内
```
cd xxx/xxx/folder
```

* 返回上级目录

```dos
cd ../
```

* 有多少层级目录就写多少个`../`便返回相应的某一层

## 统计字符串出现次数

```
cat ./logs/error.iptv-meshing.log | grep 'parameter error' | wc -l
```

## 统计端口`TCP`连接状态数量

```
netstat -anp|grep 7090 | grep ESTABLISHED -c
```
* `TCP`连接状态

## 生成文件夹的目录结构

> 查看命令 `tree /?`

```
D:\>tree /?
以图形显示驱动器或路径的文件夹结构。                                                                     

TREE [drive:][path] [/F] [/A]

   /F   显示每个文件夹中文件的名称。                                                                     
   /A   使用 ASCII 字符，而不使用扩展字符。                                                              
```

> 生成当前目录中指定文件夹下的目录树,列出文件夹中文件的名称

```
# 生成文件命令
D:\Code\sts-work-workspace\iptvMeshing>tree ./src /f >iptv_meshing.txt

# 生成文件的内容
卷 Development 的文件夹 PATH 列表
卷序列号为 8C4A-90AB
D:\SRC
├─main
│  ├─java
│  │  └─com
│  │      └─sanss
│  │          │  Application.java
│  │          │  
│  │          ├─config
│  │          │      GeocodingProperties.java
│  │          │      HikariDataSourceConfig.java
│  │          │      RestTemplateConfig.java
│  │          │      
│  │          ├─controller
│  │          │      AdUserController.java
│  │          │      Decetion.java
│  │          │      OrderController.java
│  │          │      
│  │          ├─entity
│  │          │  │  AdUserGridDO.java
│  │          │  │  OrderDO.java
│  │          │  │  
│  │          │  └─dto
│  │          │          AdUserDTO.java
│  │          │          GDPoint.java
│  │          │          
│  │          ├─repository
│  │          │      AdUserRepository.java
│  │          │      OrderRepository.java
│  │          │      
│  │          ├─service
│  │          │  │  GeocodeService.java
│  │          │  │  OrderService.java
│  │          │  │  SynchronizeInformationService.java
│  │          │  │  
│  │          │  └─impl
│  │          │          GeocodeServiceImpl.java
│  │          │          OrderServiceImpl.java
│  │          │          SynchronizeInformationServiceImpl.java
│  │          │          
│  │          └─util
│  │              │  ArrayUtil.java
│  │              │  HttpServletRequestReader.java
│  │              │  
│  │              └─geocode
│  │                      GeoCodingUtil.java
│  │                      
│  ├─resources
│  │      application.yml
│  │      geocode.properties
│  │      logback-spring.xml
│  │      
│  └─webapp
└─test
    └─java
        └─com
            └─sanss
                │  ApplicationTest.java
                │  
                ├─config
                │      HikariDataSourceConfigTest.java
                │      
                └─util
                        GeoCodingUtilTest.java
```

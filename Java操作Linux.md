---
title: Java操作Linux
toc: true
layout: blog
type: "blog"
categories:
  - Blog
  - Java
tags:
  - Java
date: 2019-11-07 10:12:17
---

{{ title }}

<!-- more -->

# `Java`操作`Linux`

### 使用`java.lang.Runtime`操作运行脚本

```java
import lombok.extern.slf4j.Slf4j;

/**
  * @Title callCloseCommand
  * @Description 执行关闭程序脚本
  * @Param
  * @param
  * @return void
  **/
private void callCloseCommand() {
  String command = "./server.sh stop";
  log.error("高德地理编码API失效,请更新IP地址", command);
  try {
    Runtime.getRuntime().exec(command);

  } catch (Exception e) {
    log.error("Linux 命令执行错误,{}", command);
    e.printStackTrace();
  }
}
```

* 注: 也可以根据实际情况调整`command`内容,更改`linux`语句。例如:`./server.sh restart`
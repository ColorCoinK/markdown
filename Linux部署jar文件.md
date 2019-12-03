---
title: Linux部署jar文件
toc: true
layout: blog
categories:
  - Blog
  - Linux
tags:
  - Blog
  - Linux
  - Shell
date: 2019-04-24 09:46:49
---
将`SpringBoot`生成的`jar`文件,在`Linux`启动。
<!-- more -->

> `Tips`

  * `windows`下的换行符为`CRLF`,在`Linux`中无法运行。如果在`windows`中编写,需要将换行符改为`LF`

  * 本博客中的`server.sh`与`xx.jar`在同级目录,所以将`cd folder`的命令注释。看官可依据自身实际情况调整,希望能解决你的问题。

# 脚本文件

```sh
#description:启动重启server服务
#端口号,根据端口号确定PID
PORT=7091
#启动命令所在目录
# HOME=/opt/xx
#启动jar文件名,需要修改为自己jar包的名字
APP_NAME=xx.jar

#查询出监听了PORT端口TCP协议的程序
pid=`netstat -anp|grep ${PORT}|awk '{printf $7}'|cut -d/ -f 1`

#检查程序是否在运行
is_exist(){
    #如果不存在返回 1 ,存在返回 0
    if [ -z "${pid}" ]; then
      return 1
    else
      return 0
    fi
}

start(){
    is_exist
    if [ $? -eq "0" ]; then
        echo "${APP_NAME} is already running. pid=${pid}."
    else
        # 进去命令所在目录
        # cd ${home} 
        # 启动jar文件,设置激活文件为prod。
        nohup java -jar $APP_NAME --spring.profiles.active=prod &
        echo "start at port:$PORT"
    fi
}

stop(){
    is_exist
    if [ $? -eq "0" ]; then
      kill -9 $pid
      echo ${APP_NAME}" is runnig on ${PORT} : ${pid}, execute kill - 9 ${pid}"
    else
      echo "${APP_NAME} is not running."
    fi
}

status(){
    is_exist
    if [ $? -eq "0" ]; then
        echo "${APP_NAME} is  runnig on port:$PORT, pid is ${pid}"
    else
        echo "${APP_NAME} is not runnig."
    fi
}

restart(){
    stop
    start
}

case "$1" in
    "start")
        start
    ;;
    "stop")
        stop
    ;;
    "restart")
      restart
    ;;
    "status")
        status
    ;;
    *)
      usage
    ;;
esac

exit 0
```

> `./server.sh`无法执行

```sh
chmod 777 xx.sh
```

> 可执行命令行

* `./server.sh start` 启动命令,控制台输出是否启动成功
* `./server.sh stop`  关闭命令,控制台输出是否关闭成功
* `./server.sh restart` 应用先关闭,后启动。控制台输出是否成功执行该操作

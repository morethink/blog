---
title: Windows和Linux查看端口和杀死进程
date: 2017-7-30
tags:
categories: Linux
---

本文介绍Windows和Linux下查看端口和杀死进程的知识。

<!-- more -->

# Windows下查看端口和杀死进程

1. 查看占用端口号的进程号：`netstat –ano | findstr "指定端口号"`
2. 通过进程号杀死进程：`taskkill /pid 进程号`
    - 通过进程号强制杀死进程：`taskkill /f /pid 进程号`
3. 通过进程号查看进程 `tasklist | findstr "进程号"`

# Linux下查看端口和杀死进程

- **Linux下查看端口号所使用的进程号**
    - 使用lsof： `lsof -i:端口号`，lsof需要拥有该进程的权限，方可以查看占用情况。这时可以使用下面这种方法。
    - 使用netstat：`netstat -anp|grep port`
- **杀死进程**
    - `kill -9 PID`
    - Linux下还提供了一个killall命令，可以直接使用进程的名字而不是进程标识号，例如： `killall -9 name`

- **通过端口号查看进程号**
    1. 查看程序对应进程号：`ps -ef | grep 进程名`
    1. 查看进程号所占用的端口号
        - REDHAT: `netstat -nltp|grep pid`
        - ubuntu: `netstat -anp|grep pid`

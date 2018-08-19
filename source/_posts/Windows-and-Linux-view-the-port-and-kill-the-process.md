---
title: Windows、Linux及Mac查看端口和杀死进程
date: 2017-07-30
tags:
categories: Linux
---

本文介绍如何在Windows、Linux及Mac下查看端口和杀死进程。

<!-- more -->

# Windows下查看端口和杀死进程

1. 查看占用端口号的进程号：`netstat –ano | findstr "指定端口号"`
2. 通过进程号杀死进程：`taskkill /pid 进程号`
    - 通过进程号强制杀死进程：`taskkill /f /pid 进程号`
3. 通过进程号查看进程 `tasklist | findstr "进程号"`

# Linux下查看端口和杀死进程

1. Linux下查看端口号所使用的进程号：`netstat -anp|grep port`
2. 杀死进程
    - `kill -9 PID`
    - Linux下还提供了一个killall命令，可以直接使用进程的名字而不是进程标识号，例如： `killall -9 name`

3. 通过端口号查看进程号
    1. 查看程序对应进程号：`ps -ef | grep 进程名`
    2. 查看进程号所占用的端口号
        - REDHAT: `netstat -nltp|grep pid`
        - ubuntu: `netstat -anp|grep pid`

# Mac下查看端口和杀死进程

Mac下使用lsof（list open files）来查看端口占用情况，lsof 是一个列出当前系统打开文件的工具。

使用 `-i` 查看某个端口是否被占用，如：
```
lsof -i:3000
```
如果端口被占用，则会返回相关信息，如果没被占用，则不返回任何信息。

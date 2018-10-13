---
title: Jenkins在Docker中执行Shell出现乱码
date: 2018-10-10
tags:
    - Jenkins
    - Docker
categories: Linux
---

一直以来都觉得在图片下面添加一个标题可以更加清晰的表示这张图片的含义，可是博客园原生并不支持这种渲染方式，再加上博客园可以自己写js来更改主题，于是通过搜索资料完成给博客园图片添加标题的功能。

<!-- more -->

问题说明：
Jenkins里直接执行shell脚本，会出现中文乱码的问题。但是单独执行shell脚本又是没问题。这个怎么办呢？

解决办法：
在shell脚本开始的时候加上命令:

export LANG="en_US.UTF-8"

中文乱码的问题就解决了。 此问题是crontab执行shell脚本乱码是同一个问题。


问题原因：
此问题是因此执行定时任务时没有去获取系统的环境变量，导致了中文乱码。

[Jenkins执行shell无法读取环境变量问题](https://my.oschina.net/mrpei123/blog/1821380)

[交互式SHELL和非交互式SHELL、登录SHELL和非登录SHELL的区别](https://blog.csdn.net/wisgood/article/details/52043522)


# 常见的shell变量

```
PATH：决定了shell将到哪些目录中寻找命令或程序
HOME：当前用户主目录
MAIL：是指当前用户的邮件存放目录。
SHELL：是指当前用户用的是哪种Shell。
HISTSIZE：是指保存历史命令记录的条数
LOGNAME：是指当前用户的登录名。
HOSTNAME：是指主机的名称，许多应用程序如果要用到主机名的话，通常是从这个环境变量中来取得的。
LANG/LANGUGE：是和语言相关的环境变量，使用多种语言的用户可以修改此环境变量。
PS1：是基本提示符，对于root用户是#，对于普通用户是$。
PS2：是附属提示符，默认是">"。
```

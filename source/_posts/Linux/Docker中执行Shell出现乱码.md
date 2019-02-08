---
title: Docker中执行Shell出现乱码
date: 2018-10-14
tags: Docker
categories: Linux
---

# 问题描述

最近遇到一个问题：
执行命令
```shell
docker exec f4af9b sh -c 'bash /tmp/build.sh'
```
在docker中执行shell，会出现中文乱码的问题。但是在docker容器中单独执行shell脚本却没有出现乱码。查看环境变量存在`LANG=en_US.UTF-8`，因此从原理上来说是不应该出现乱码的。

但是既然出现了乱码，那么`LANG=en_US.UTF-8`应该就没有读取到，于是在 `build.sh`中运行`env`命令，发现通过`docker exec f4af9b sh -c 'bash /tmp/build.sh'`方式没有`LANG=en_US.UTF-8`环境变量，那么原因是什么？


<!-- more -->

# 原因定位

原因如下：
`docker exec f4af9b sh -c 'bash /tmp/build.sh'` 对于docker 容器来说是非登录和非交互式shell，这样就不会读取某些配置文件，导致`LANG=en_US.UTF-8`没有加载成功。

# Linux Shell

下面介绍一下Linux交互式和非交互式shell、登录和非登录shell之间的区别。

- **交互式shell（interactive shell）和非交互式shell（non-interactive shell）**：
    - 交互式的shell会有一个输入提示符，并且它的标准输入、输出和错误输出都会显示在控制台上。这种模式也是大多数用户非常熟悉的：登录、执行一些命令、退出。当你退出后，shell也终止了。  
    - 非交互式shell是`bash script.sh`这类的shell。在这种模式下，shell不与你进行交互，而是读取存放在文件中的命令，并且执行它们。当它读到文件的结尾EOF，shell也就终止了。

- **登录式shell（login shell）和非登陆式shell（no-login shell）**：
    - 需要输入用户名和密码的shell就是登陆式shell。因此通常不管以何种方式登陆机器后用户获得的第一个shell就是login shell。不输入密码的ssh是公钥打通的，某种意义上说也是输入密码的。
    - 非登陆式的就是在登陆后启动bash等，即不是远程登陆到主机这种。

对于常用环境变量设置文件，整理出如下加载情况表：

| 文件 	| 非交互+登陆式 	| 交互+登陆式 	| 交互+非登陆式 	| 非交互+非登陆式 	|
|:---------------:	|:-------------:	|:-----------:	|:-------------:	|:---------------:	|
| /etc/profile 	| 加载 	| 加载 	| - 	| - 	|
| /etc/bashrc 	| 加载 	| 加载 	| - 	| - 	|
| ~/.bash_profile 	| 加载 	| 加载 	| - 	| - 	|
| ~/.bashrc 	| 加载 	| 加载 	| 加载 	| - 	|
| BASH_ENV 	| - 	| - 	| - 	| 加载 	|


执行脚本，如`bash script.sh`是属于non-login + non-interactive。

# 解决思路
因而，执行命令`docker exec f4af9b sh -c 'bash /tmp/build.sh'`对于docker容器来说是属于non-login + non-interactive。

将上面的`bash /tmp/build.sh`改为`bash --login /tmp/build.sh`变为登录shell，就可以读取/etc/profile和~/.bash_profile等文件。

或者在执行`bash /tmp/build.sh`时在`build.sh`加入`export LANG="en_US.UTF-8"`手动设置。

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

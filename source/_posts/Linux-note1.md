---
title: Linux学习笔记一
date: 2017-7-30
tags: Linux
categories: Linux
---

本文记录了Linux中常用的一些东西。

**命令生效顺序**

1. 第一顺位执行绝对路径或者相对路径的命令
2. 第二顺位执行别名
3. 第三顺位执行Bash的内部命令
4. 第四顺位执行按照`$PATH`环境变量设置定义的目录顺序的第一个命令

<!-- more -->

# 脚本执行方式

当我们写完一个脚本的时候，它是不可以直接运行的。

1. 通过Bash调用执行脚本 `# bash hello.sh`
2. 赋予执行权限，直接运行
    - `# chmod 755 hello.sh`
    - 相对路径 `# ./hello.sh` 或者 绝对路径 `# /root/hello.sh`

# 设置PATH环境变量

1. 直接用export命令 `# export PATH=$PATH:/usr/local/src/node-v0.10.24/node_modules/node-sass/bin`
2. 修改profile文件：
    - `# vi /etc/profile`
    - 然后在文件里面加入`export PATH=$PATH:/usr/local/src/node-v0.10.24/node_modules/node-sass/bin`
    - 执行命令 `# source /etc/profile` 生效

3. 修改.bashrc文件

    - `# vi /root/.bashrc`
    - 在里面加入 `export PATH=$PATH:/usr/local/src/node-v0.10.24/node_modules/node-sass/bin`
    - 执行命令`# source /root/.bashrc` 生效

如果修改了`/etc/profile`，那么编辑结束后执行`# source profile`(`# source /etc/profile`) 或 执行点命令 `# . ./profile`,PATH的值就会立即生效了。 这个方法的原理就是再执行一次/etc/profile shell脚本，注意如果用sh /etc/profile是不行的，因为sh是在子shell进程中执行的，即使PATH改变了也不会反应到当前环境中，但是source是在当前 shell进程中执行的，所以我们能看到PATH的改变

**PATH环境变量设置错误导致命令失效**
1. 如果是通过命令行设置的，可以通过重启或者离开本次登录(退出本次shell)
2. 如果是因为修改配置文件导致无法使用，可以执行`# export PATH=/usr/bin:/usr/sbin:/bin:/sbin:/usr/X11R6/bin`命令暂时使用这些目录下的命令，如 `vi`,`ls`,`cd`，从新修改配置文件。

# 搜索命令

## 1 find

1. find是最常见和最强大的查找命令，你可以用它找到任何你想找的文件。（避免大范围的搜索，会非常浪费系统资源，建议不在直接在"/"目录下搜索）
2. 格式：find [搜索范围] [搜索条件]
3. 例：

    - find /home -name 文件名
    - find /home -iname 文件名 (不区分大小写)
    - find /home -user 文件名 (所有者文件)
    - find /home -nouser 文件名(没有所属者的文件，liunx中，每个文件都有所属者，如果没有，那么一般都是垃圾文件，但还是有特例的，比如内核产生的文件，就没有所属者，一般在proc和sys目录下；还有外来文件，也就是U盘拷入的文件也会忽略所有者。)
    - find . -type f -mmin -10 搜索当前目录中，所有过去10分钟中更新过的普通文件。如果不加-type f参数，则搜索普通文件+特殊文件+目录。
    - find 目录 -size 文件大小 (注意：文件大小用小写k和大写M。)

**如果什么参数也不加，find默认搜索当前目录及其子目录，并且不过滤任何结果（也就是返回所有文件），将它们全都显示在屏幕上。**

## 2 locate

locate命令其实是"find -name"的另一种写法，但是要比后者快得多，原因在于它不搜索具体目录，而是搜索一个数据库（/var/lib/locatedb），这个数据库中含有本地所有文件信息。Linux系统自动创建这个数据库，并且每天自动更新一次，所以使用locate命令查不到最新变动过的文件。为了避免这种情况，可以在使用locate之前，先使用**updatedb**命令，手动更新数据库。 locate命令的使用实例：

  - `# locate /etc/sh`,搜索etc目录下所有以sh开头的文件。
  - `# locate ~/m`,搜索用户主目录下，所有以m开头的文件。
  - `# locate -i ~/m`,搜索用户主目录下，所有以m开头的文件，并且忽略大小写。

## 3 whereis

whereis命令只能用于**程序名**的搜索，而且只搜索二进制文件（参数-b）、man说明文件（参数-m）和源代码文件（参数-s）。如果省略参数，则返回所有信息。 whereis命令的使用实例： `# whereis grep`

## 4 which

which命令的作用是，**在PATH变量指定的路径中**，搜索某个系统命令的位置，并且返回第一个搜索结果。也就是说，使用which命令，就可以看到某个系统命令是否存在，以及执行的到底是哪一个位置的命令。 which命令的使用实例： `# which grep`

## grep

搜索文件中的信息，一般为字符串，与正则表达式结合使用 grep [选项] 字符串 文件名 （字符串使用 "" 包围，结果为行记录） -i 忽略大小写 -v 排除指定字符串

**find与grep的区别**
- find：在 系统 中搜索符合条件的 文件名，使用 通配符（完全）匹配
- grep：在 文件 当中搜索符合条件的 字符串，使用 正则表达式 （包含）匹配

#

**参考文档**

1. [Linux的五个查找命令](http://www.ruanyifeng.com/blog/2009/10/5_ways_to_search_for_files_using_the_terminal.html)

---
title: Linux常用命令
date: 2017-07-30
tags:
categories: Linux
---

本文记录一些在Linux系统下比较常用的命令。

首先介绍下Linux下命令生效的顺序：
1. 第一顺位：执行绝对路径或者相对路径的命令
2. 第二顺位：执行别名
3. 第三顺位：执行Bash的内部命令
4. 第四顺位：执行按照`$PATH`环境变量设置定义的目录顺序的第一个命令
<!-- more -->

# cp
该命令用于复制文件，copy之意，它还可以把多个文件一次性地复制到一个目录下，它的常用参数如下：

- -a：将文件的特性一起复制  
- -p：连同文件的属性一起复制，而非使用默认方式，与-a相似，常用于备份  
- -i：若目标文件已经存在时，在覆盖时会先询问操作的进行  
- -r：递归持续复制，用于目录的复制行为  
- -u：目标文件与源文件有差异时才会复制  

例：
- `cp source dest` 复制文件
- `cp -r sourceFolder targetFolder` 递归复制整个文件夹


# mv

该命令用于移动文件、目录或更名，move之意，它的常用参数如下：
- -f：force强制的意思，如果目标文件已经存在，不会询问而直接覆盖  
- -i：若目标文件已经存在，就会询问是否覆盖  
- -u：若目标文件已经存在，且比目标文件新，才会更新  

注：该命令可以把一个文件或多个文件一次移动一个文件夹中，但是最后一个目标文件一定要是“目录”。

例：
- `mv file1 file2 file3 dir`  把文件file1、file2、file3移动到目录dir中  
- `mv file1 file2` 把文件file1重命名为file2  

# rm
该命令用于删除文件或目录，remove之间，它的常用参数如下：

- -f ：就是force的意思，忽略不存在的文件，不会出现警告消息  
- -i ：互动模式，在删除前会询问用户是否操作  
- -r ：**递归删除，最常用于目录删除，它是一个非常危险的参数**  

例如：

- `rm -i file` 删除文件file，在删除之前会询问是否进行该操作  
- `rm -fr dir`  强制删除目录dir中的所有文件


# ps
该命令用于将某个时间点的进程运行情况选取下来并输出，process之意，它的常用参数如下：

- -A ：所有的进程均显示出来  
- -a ：不与terminal有关的所有进程  
- -u ：有效用户的相关进程  
- -x ：一般与a参数一起使用，可列出较完整的信息  
- -l ：较长，较详细地将PID的信息列出  


其实我们只要记住ps一般使用的命令参数搭配即可，它们并不多，如下：
- `ps aux` 查看系统所有的进程数据  
- `ps ax`  查看不与terminal有关的所有进程  
- `ps -lA`  查看系统所有的进程数据  
- `ps axjf` 查看连同一部分进程树状态

查看某个进程：
- `ps –ef|grep tomcat` 查看所有有关tomcat的进程
- `ps -ef|grep --color java` 高亮要查询的关键字


# kill
该命令用于向某个工作（jobnumber）或者是某个PID（数字）传送一个信号，它通常与ps和jobs命令一起使用，它的基本语法如下：
`kill -signal PID `
signal的常用参数(最前面的数字为信号的代号，使用时可以用代号代替相应的信号)如下：
- 1：SIGHUP，启动被终止的进程  
- 2：SIGINT，相当于输入ctrl+c，中断一个程序的进行  
- 9：SIGKILL，强制中断一个进程的进行  
- 15：SIGTERM，以正常的结束进程方式来终止进程  
- 17：SIGSTOP，相当于输入ctrl+z，暂停一个进程的进行  


例如：
- 以正常的结束进程方式来终于第一个后台工作，可用jobs命令查看后台中的第一个工作进程  
`kill -SIGTERM %1` 或者 `kill -15 %1`
- 重新改动进程ID为PID的进程，PID可用ps命令通过管道命令加上grep命令进行筛选获得  
`kill -SIGHUP PID` 或者 `kill -1 PID`
- 终止线程号位19979的进程
`kill -9 19979`

# killall

该命令用于向一个命令启动的进程发送一个信号，它的一般语法如下：

`killall [-iIe] [command name]`  
它的参数如下：

- -i ：交互式的意思，若需要删除时，会询问用户  
- -e ：表示后面接的command name要一致，但command name不能超过15个字符  
- -I ：命令名称忽略大小写  

例如：  
`killall -SIGHUP syslogd` 重新启动syslogd  

# file

该命令用于判断接在file命令后的文件的基本数据，因为在Linux下文件的类型并不是以后缀为分的，所以这个命令对我们来说就很有用了，它的用法非常简单，基本语法如下：
`file filename`  
例如：  
`file ./test`

# cat命令
该命令用于查看文本文件的内容，后接要查看的文件名，通常可用管道与more和less一起使用，从而可以一页页地查看数据。例如：

`cat text | less`  查看text文件中的内容  
注：这条命令也可以使用less text来代替  

# chgrp命令

该命令用于改变文件所属用户组，它的使用非常简单，它的基本用法如下：

`chgrp [-R] dirname/filename `
-R ：进行递归的持续对所有文件和子目录更改  

例如：  
`chgrp users -R ./dir ` 递归地把dir目录下中的所有文件和子目录下所有文件的用户组修改为users  

# chown命令

该命令用于改变文件的所有者，与chgrp命令的使用方法相同，只是修改的文件属性不同，不再详述。

# chmod命令
该命令用于改变文件的权限，一般的用法如下：

`chmod [-R] xyz 文件或目录 `
-R：进行递归的持续更改，即连同子目录下的所有文件都会更改  
同时，chmod还可以使用u（user）、g（group）、o（other）、a（all）和+（加入）、-（删除）、=（设置）跟rwx搭配来对文件的权限进行更改。

例如：  
`chmod 755 file`  把file的文件权限改变为-rxwr-xr-x，r表示读、w表示写、x表示可执行。
`chmod g+w file` 向file的文件权限中加入用户组可写权限。
`chmod +x file` 让文件可执行。

权限代号：
- r：读权限，用数字4表示
- w：写权限，用数字2表示
- x：执行权限，用数字1表示

# 关于搜索的命令

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

  - `locate /etc/sh`,搜索etc目录下所有以sh开头的文件。
  - `locate ~/m`,搜索用户主目录下，所有以m开头的文件。
  - `locate -i ~/m`,搜索用户主目录下，所有以m开头的文件，并且忽略大小写。

## 3 whereis

whereis命令只能用于**程序名**的搜索，而且只搜索二进制文件（参数-b）、man说明文件（参数-m）和源代码文件（参数-s）。如果省略参数，则返回所有信息。 whereis命令的使用实例： `whereis grep`

## 4 which

which命令的作用是，**在PATH变量指定的路径中**，搜索某个系统命令的位置，并且返回第一个搜索结果。也就是说，使用which命令，就可以看到某个系统命令是否存在，以及执行的到底是哪一个位置的命令。 which命令的使用实例： `which grep`

# grep

搜索文件中的信息，一般为字符串，与正则表达式结合使用 grep [选项] 字符串 文件名 （字符串使用 "" 包围，结果为行记录） -i 忽略大小写 -v 排除指定字符串

**find与grep的区别**：
- find：在 系统 中搜索符合条件的 文件名，使用 通配符（完全）匹配
- grep：在 文件 当中搜索符合条件的 字符串，使用 正则表达式 （包含）匹配

# time

time 可以统计命令执行的时间，包括程序的实际运行时间(real time)，以及程序运行在用户态的时间(user time)和内核态的时间(sys time)。
例如：`time git pull` 则是统计更新代码花费多长时间

# alias

alias命令用来设置指令的别名。我们可以使用该命令可以将一些较长的命令进行简化。使用alias时，用户必须使用单引号''将原来的命令引起来，防止特殊字符导致错误。

alias命令的作用只局限于该次登入的操作。若要每次登入都能够使用这些命令别名，则可将相应的alias命令存放到bash的初始化文件`/etc/bashrc`中。

alias 的基本使用方法为：`alias 新的命令='原命令 -选项/参数'`
例如：`alias l=‘ls -lsh'`将重新定义ls命令，现在只需输入l就可以列目录了。直接输入 alias 命令会列出当前系统中所有已经定义的命令别名。

要删除一个别名，可以使用 unalias 命令，如 unalias l。

**查看系统已经设置的别名：**

```shell
alias egrep='egrep --color'
alias fgrep='fgrep --color'
alias grep='grep --color'
alias hpush='bash /Users/liwenhao/Desktop/github/hpush.sh'
alias ll='ls -l'
alias ls='ls -F --show-control-chars --color=auto'
alias vi='vim'
```
# 执行多个命令

- 后一个命令依赖于前一个命令的输出，可以是用管道(|)
`ls | wc -l` :当前目录文件个数
- 后一个命令必须等前一个命令运行成功后在运行，可以使用双与号(&&)
`aa && ls` ：只运行aa，ls不运行
- 后一个命令必须等前一个命令运行完，不关心是否成功，使用单与号(&)
`aa & ls` ：aa和ls都运行，但是ls必须等aa运行完。
- 并行执行多个命令，使用两个竖号(||)
`aa || ls`：aa和ls并行执行，互不影响。

# 执行shell脚本

当我们写完一个脚本的时候，它是不可以被直接运行的。我们可以通过：
1. 通过Bash调用执行脚本 ：`bash hello.sh`
2. 首先赋予执行权限：`chmod 755 hello.sh`，然后就可以通过相对路径 `./hello.sh` 或者通过绝对路径 ` /root/hello.sh`来执行。


# PATH环境变量

下面的第一种方式关机后会失效，相当于临时环境变量，后面两种都是写进了系统配置，因此会永久生效。
1. 直接用export命令 `export PATH=$PATH:/usr/local/src/node-v0.10.24/node_modules/node-sass/bin`
2. 修改profile文件：
    - `vi /etc/profile`
    - 然后在文件里面加入`export PATH=$PATH:/usr/local/src/node-v0.10.24/node_modules/node-sass/bin`
    - 执行命令 `source /etc/profile` 生效

3. 修改.bashrc文件
    - `vi /root/.bashrc`
    - 在里面加入 `export PATH=$PATH:/usr/local/src/node-v0.10.24/node_modules/node-sass/bin`
    - 执行命令`source /root/.bashrc` 生效

如果修改了`/etc/profile`，那么编辑结束后执行` source profile`(`source /etc/profile`) 或 执行点命令 `. ./profile`，PATH的值才会立即生效，不然就只有等下一次开机。

这个方法的原理就是再执行一次/etc/profile shell脚本，注意如果用sh /etc/profile是不行的，因为sh是在子shell进程中执行的，即使PATH改变了也不会反应到当前环境中，但是source是在当前 shell进程中执行的，所以我们能看到PATH的改变。

有时我们可能不小心将PATH环境变量设置错误导致命令失效，可以参考下面两种方式解决。
1. 如果是通过命令行设置的，可以通过重启或者离开本次登录(退出本次shell)
2. 如果是因为修改配置文件导致无法使用，可以执行`export PATH=/usr/bin:/usr/sbin:/bin:/sbin:/usr/X11R6/bin`命令暂时使用这些目录下的命令，如 `vi`,`ls`,`cd`，从新修改配置文件。


# 命令行快捷键

1. 光标移动
    - **Ctrl + f – 向右移动一个字符，当然多数人用→**
    - **Ctrl + b – 向左移动一个字符， 多数人用←**
    - ESC + f – 向右移动一个单词，MAC下建议用ALT + →
    - ESC + b – 向左移动一个单词，MAC下建议用ALT + ←
    - **Ctrl + a – 跳到行首**
    - **Ctrl + e – 跳到行尾**
2. 删除
    - Ctrl + d – 向右删除一个字符
    - **Ctrl + h – 向左删除一个字符**
    - **Ctrl + u – 删除当前位置字符至行首（输入密码错误的时候多用下这个）**
    - Ctrl + k – 删除当前位置字符至行尾
    - Ctrl + w – 删除从光标到当前单词开头
3. 命令切换
    - **Ctrl + p – 上一个命令，也可以用↑**
    - **Ctrl + n – 下一个命令，也可以用↓**
4. 其他快捷键
    - Ctrl + y – 插入最近删除的单词
    - Ctrl + c – 终止操作
    - Ctrl + d – 当前操作转到后台
    - **Ctrl + l – 清屏 （有时候为了好看）**
    - ctrl + z - 把命令放入后台，这个命令谨慎使用
    - **ctrl + r - 历史命令搜索**



**参考文档**：
1. [Linux的五个查找命令](http://www.ruanyifeng.com/blog/2009/10/5_ways_to_search_for_files_using_the_terminal.html)
2. [初窥Linux 之 我最常用的20条命令](https://blog.csdn.net/ljianhui/article/details/11100625)

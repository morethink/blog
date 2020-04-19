---
title: CentOS7  MySQL5.7安装、JDK8安装、Tomcat安装、Maven热部署
date: 2017-4-25
tags: [CentOS7,Tomcat,MySQL,Maven,MySQL5.7]
categories: Linux
---

本文介绍了CentOS7 下MySQL5.7、Java、Tomcat、Maven热部署等服务器环境的搭建和调试过程。

学生服务器资源获取方法：
1. [云+校园计划 - 腾讯云](https://www.qcloud.com/act/campus)
2. 阿里云云翼计划
3. github 学生包，里面有Digital Ocean 50美元的VPS可用


已经将所需要的工具(Xshell,Xftp、FileZilla等sftp上传工具，jdk-8u101-linux-x64.tar.gz和apache-tomcat-9.0.0.M10.tar.gz)上传至百度云 http://pan.baidu.com/s/1qYRms8G

<!-- more -->


# MySQL5.7

## 安装MySQL5.7

CentOS 7的yum源中貌似没有正常安装mysql时的mysql-sever文件，需要去官网上下载

```shell
# wget https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
# rpm -ivh mysql57-community-release-el7-11.noarch.rpm
# yum install mysql-community-server
# 成功安装之后启动mysql服务
# systemctl start mysqld.service
# 初次安装mysql是root账户是没有密码的，需要我们自己设置密码
# mysql -uroot
# mysql>set password = password("你的密码");
# mysql>flush privileges;
# mysql> exit
```

## ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)

安装完MySQL5.7后 执行 `mysql -uroot`首次登陆会出现此问题，原因是 MySQL5.7设置了临时密码。
```shell
$sudo cat /var/log/mysqld.log  | grep password
2017-04-14T11:45:46.950302Z 1 [Note] A temporary password is generated for root@localhost: NS_E#kj!E5lJ
$mysql -uroot -pNS_E#kj!E5lJ
```

## ERROR 1819 (HY000): Your password does not satisfy the current policy requirements

原来MySQL5.6.6版本之后增加了密码强度验证插件validate_password，相关参数设置的较为严格。使用了该插件会检查设置的密码是否符合当前设置的强度规则，若不满足则拒绝设置。影响的语句和函数有：`create user,grant,set password,password(),old password`。简单点说就是系统认为密码太简单，设置复杂点就行了。如 `!QAZ2wsx`。

## 设置MySQL远程连接

进入MySQL后通过
```
mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '你的密码' WITH GRANT OPTION;
```
可以允许任意主机访问你在服务器搭建的MySQL，可以吧`%`更换为你需要允许的IP和主机。


但是，在我进行远程连接的时候发生如下错误
!["MySQL 2003"](https://images.morethink.cn/mysql-2003-error.png "MySQL 2003")
可以根据下面进行排错
1. 服务器是否可以访问，是否能ping 通
2. 安全组及端口号是否打开或者被占用
3. 服务器是否运行或者关闭只允许本机访问
4. 重启试试

当我排除上面3中情况后，发现属于第四中，可能是因为还没有立即生效。

## 解决MySQL中文乱码问题

在我建立数据时发现中文无法插入，于是查看使用`show variables like 'character%';`
如图所示，发现默认不是utf-8，如图:
!["MySQL中文乱码"](https://images.morethink.cn/msql-chinese-fail.jpg "MySQL中文乱码")

于是通过在CentOS7中修改文件/usr/share/mysql/my-default.cnf，在[mysqld]，[mysql]，[client]下分别添加如下内容:
```
[mysqld]
collation-server = utf8_unicode_ci
init-connect='SET NAMES utf8'
character_set_server = utf8

[mysql]

default-character-set=utf8

[client]

default-character-set=utf8

```

修改完成后，重启mysql服务，systemctl restart mysql，然后进入mysql，再次使用`show variables like 'character%';`命令查看，如图:
![](https://images.morethink.cn/msql-chinese-success.jpg)
发现一样，说明utf-8可以使用，但是无法插入中文，于是猜测可能是系统原生不支持utf-8，更改系统编码后发现果然如此。
```shell
# cat /etc/locale.conf
LANG="en_US.UTF-8"
# vim /etc/locale.conf
# cat /etc/locale.conf
LANG="zh_CN.UTF-8"
```
于是更改为

```shell
LANG="zh_CN.UTF-8"
LANGUAGE="zh_CN.UTF-8"
SUPPORTED="zh_CN.UTF-8:zh_CN:zh:en_US.UTF-8:en_US:en"
SYSFONT="lat0-sun16"
```

执行 `source /etc/locale.conf`或者`重启服务器`之后，再次重启MySQL服务。

如图，发现可以插入中文。

![](https://images.morethink.cn/mysql-chinese-success-result.jpg)


## 解决MySQL忘记密码

通过跳过权限安全检查设置新密码。

1. 首先检查mysql服务是否启动，若已启动则先将其停止服务，可在开始菜单的运行，使用命令： `net stop mysql`
，然后打开第一个cmd1窗口，切换到mysql的bin目录，运行命令：`mysqld --defaults-file="C:\Program Files\MySQL\MySQL Server 5.5\my.ini" --console --skip-grant-tables`，将命令中的MySql版本更换你的版本。
**该命令通过跳过权限安全检查，开启mysql服务，这样连接mysql时，可以不用输入用户密码**。
此时已经开启了mysql服务了！
**这个窗口保留不关闭**。
2. 打开第二个cmd2窗口，连接mysql
    - 输入命令：`mysql -u root -p`
      出现： `Enter password:` ，在这里直接回车，不用输入密码。 然后就就会出现登录成功的信息。
    - 使用命令切换到mysql数据库：`use mysql;`
    - 使用命令更改root密码：`UPDATE user SET Password=PASSWORD('newpassword') where USER='root';`
    - 刷新权限：`FLUSH PRIVILEGES;`
    - 然后退出，重新登录：`quit`
    - 重新登录： 可以关掉之前的cmd1 窗口了。
3. 然后用`net start mysql` 启动服务
    - 登录：`mysql -u root -p`
    - 出现输入密码提示，输入新的密码即可登录：`Enter password: ***********`

显示登录信息： 成功  就一切ok了

# Java环境配置

## 环境准备

通过`uname -r`判断系统是多少位
- 64位 ： 出现x86_64
- 32位 ： 出现i686或i386


## 安装Java JDK8.0

1. 建立Java目录，存放Java和Tomcat
    - `cd /usr/local/`
    - `mkdir Java`
    - `cd Java`
2. 使用FileZilla将下载好的jdk-8u101-linux-x64.tar.gz 和 apache-tomcat-9.0.0.M10.tar.gz上传至Java目录下(传送的国外服务器很慢,国内几乎是国外的十倍，但是也只有两三百KB，也可能是电脑问题)
3. 将上传的jdk解压，然后重命名为jdk
    - `tar -zxv -f  jdk-8u101-linux-x64.tar.gz`
    - `mv jdk1.8.0_101  jdk`
    - `cd jdk`
4. 配置环境变量Environment=JAVA_HOME=/usr/local/Java/jdk
      1. `vim /etc/profile`
      2. 打开之后按键盘（i）进入编辑模式,将下面的内容复制到底部
    ```
    JAVA_HOME=/usr/local/Java/jdk
    PATH=$JAVA_HOME/bin:$PATH
    CLASSPATH=$JAVA_HOME/jre/lib/ext:$JAVA_HOME/lib/tools.jar
    export PATH JAVA_HOME CLASSPATH
    ```
      3. 写完之后我们按键盘（ESC）按钮退出，然后按（:wq）保存并且关闭Vim。
      4. 使用 `source /etc/profile`命令使其立即生效
      3. 通过`java -version`验证Java是否配置成功。

# 安装Tomcat9

1. 在Java目录下解压上面一步已经上传上去的Tomcat9.0
    - `tar -zxv -f apache-tomcat-9.0.0.M10.tar.gz`
    - `mv apache-tomcat-9.0.0.M10 tomcat`
    - `cd tomcat`
2. 启动命令为 `/usr/local/Java/tomcat/bin/startup.sh`
3. 启动完成后还需开放8080端口(CentOS7这个版本的防火墙默认使用的是firewall，与之前的版本使用iptables不一样。 **关于防火墙端口可以查看后面的参考文档**)
    - `firewall-cmd --zone=public --add-port=8080/tcp --permanent`
出现success表明添加成功
    - 更新防火墙规则即可： `firewall-cmd --reload`
    - 重启防火墙 `systemctl restart firewalld.service`
4. 然后再次在浏览器中输入http://ip:8080，如果看到tomcat系统界面，说明安装成功。
5. Tomcat 8080 端口无法访问
      - 查看8080端口被那个程序占用(应该是Java) netstat -anp 然后再杀死占用进程。
      - **可能是你的服务器提供商有安全组来控制端口，你需要去提供商那里开启端口(PS：我的阿里云服务器就是必须要设置端口安全组才可以访问端口)**
6. 关闭命令为 `/usr/local/Java/tomcat/bin/shutdown.sh`

# Maven 热部署

Maven 热部署可以通过一行命令部署到本地服务器，没有问题的话就一行命令部署到正式服务器。及其方便了开发和部署。因为我的Tomcat9遇到很多问题。
可以参考 [maven自动部署到远程tomcat教程](http://www.cnblogs.com/xyb930826/p/5725340.html) 进行部署和测试。

下面是我遇到的一个错误，因为没有配置IDEA的make 导致出错。
```
[ERROR] Failed to execute goal org.apache.tomcat.maven:tomcat7-maven-plugin:2.2:deploy (default-cli) on project liwenhao: Cannot invoke Tomcat manager: Connect
ion reset by peer: socket write error -> [Help 1]
[ERROR]
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR]
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1]
```
可以通过将make如下配置
![](https://images.morethink.cn/make-maven-goal.jpg)
即可成功

**war包部署在服务器乱码**

![](https://images.morethink.cn/maven-war-messy-code.jpg)
可以通过配置如下属性，解决中文war包服务器乱码。
```
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
</properties>
```
配置完图。
![](https://images.morethink.cn/maven-war-code-success.jpg)

在我通过`mvn tomcat7:deploy`命令热部署时，会出现mysql无法连接的情况，后来在我重新进行热部署的时候，没有出现这个问题。
**猜测**
应该是我的配置文件的问题

**参考文档**
1. [centos 7 开放 80端口](http://www.centoscn.com/CentOS/config/2016/0511/7218.html)
2. [centos7 设置中文](http://www.cnblogs.com/weiok/p/5086971.html)
3. [CentOS 7下彻底卸载MySQL数据库](https://zhangzifan.com/centos-7-remove-mysql.html)
4. CentOS7 **远程访问MySQL**
    - [Centos 7 mysql 5.7 给root开启远程访问权限，修改root密码](http://blog.sina.com.cn/s/blog_5da16ee20102x47h.html)
    - **[连接Mysql提示Can’t connect to local MySQL server through socket的解决方法](http://www.aiezu.com/db/mysql_cant_connect_through_socket.html)**
5. [How To Install Apache Tomcat 8 on CentOS 7](https://www.digitalocean.com/community/tutorials/how-to-install-apache-tomcat-8-on-centos-7)
6. [windows环境中mysql忘记root密码的解决办法](http://www.cnblogs.com/linuxnotes/archive/2013/03/09/2951101.html)

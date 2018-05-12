---
title: MAC安装RabbitMQ
date: 2018-05-12
tags:
  - MQ
  - MAC
categories: 工具
photo: https://images.morethink.cn/rabbitmq-logo.png
---

# 安装

```shell
brew update
brew install rabbitmq
```
<!-- more -->
# 配置
1. 添加环境变量
    - 打开配置文件`$ vi ~/.bash_profile`
    - 添加 `export PATH=$PATH:/usr/local/sbin`
    到末尾，编辑完后:wq保存退出。
    - 使环境变量立即生效 `$ source ~/.bash_profile`
2. 启动RabbitMQ服务
上面配置完成后，需要关闭终端窗口，重新打开，然后输入下面命令即可启动RabbitMQ服务：`rabbitmq-server`
3. 登录Web管理界面
浏览器输入localhost：15672，账号密码全输入guest即可登录。


这里需要注意下，从3.3.1版本开始，RabbitMQ默认不允许远程ip登录，即只能使用localhost登录。如果希望远程登录，需要添加用户权限。

# 设置RabbitMQ远程ip登录


由于账号guest具有所有的操作权限，并且又是默认账号，出于安全因素的考虑，guest用户只能通过localhost登陆使用，并建议修改guest用户的密码以及新建其他账号管理使用rabbitmq。
这里我们以创建个test帐号，密码123456为例，创建一个账号并支持远程ip访问。

- 创建账号
```
rabbitmqctl add_user test 123456
```
- 设置用户角色
```
rabbitmqctl  set_user_tags  test  administrator
```
- 设置用户权限
```
rabbitmqctl set_permissions -p "/" test ".*" ".*" ".*"
```
- 设置完成后可以查看当前用户和角色(需要开启服务)
```
rabbitmqctl list_users
```

这是你就可以通过其他主机的访问RabbitMQ的Web管理界面了，访问方式，浏览器输入：serverip:15672。其中serverip是RabbitMQ-Server所在主机的ip。

# RabbitMQ常用操作
1. 用户管理
用户管理包括增加用户，删除用户，查看用户列表，修改用户密码。
    - 新增一个用户
    ```
    rabbitmqctl  add_user  Username  Password
    ```
    - 删除一个用户
    ```
    rabbitmqctl  delete_user  Username
    ```
    - 修改用户的密码
    ```
    rabbitmqctl  change_password  Username  Newpassword
    ```
    - 查看当前用户列表
    ```
    rabbitmqctl  list_users
    ```

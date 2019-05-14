---
title: 在vps上搭建hexo博客
date: 2019-5-15
tags:
categories: 工具
---


最近更换了服务器，需要把自己的Hexo Next重新部署到新服务器上，本文记录一下在vps上搭建hexo博客的过程。

在vps上搭建hexo博客需要下面这些工具：
1. Nginx: 用于博客展示
2. SSH：用于Git 推送
3. Git: 用于将生成的静态文件推送到vps上

本文服务器环境为CentOS 7.6
<!-- more -->

整体流程为：

![整体流程](https://images.morethink.cn/b433b4ab577d76364918879d9e150d91.png "整体流程")

# 设置SSH登录

想要完成Git推送，首先得设置SSH登录。过程如下：

```shell
# 添加hexo用户
adduser hexo
# 切换到hexo用户目录
cd /home/hexo
# 创建.ssh文件夹
mkdir .ssh
# 创建authorized_keys文件并编辑
vim .ssh/authorized_keys
# 如果你还没有生成公钥，那么首先在本地电脑中执行 cat ~/.ssh/id_rsa.pub | pbcopy生成公钥
# 再将公钥复制粘贴到authorized_keys
# 保存关闭authorized_keys后，修改相应权限
chmod 600 .ssh/authorzied_keys
chmod 700 .ssh
```

测试是否设置成功：
```shell
ssh -v hexo@服务器ip
```

# Git

## 安装Git
```shell
yum install git
```

## 配置post-update钩子
Git的钩子脚本位于版本库.git/hooks目录下，当Git执行特定操作时会调用特定的钩子脚本。当版本库通过git init或者git clone创建时，会在.git/hooks目录下创建示例脚本，用户可以参照示例脚本的写法开发适合的钩子脚本。

钩子脚本要设置为可运行，并使用特定的名称。Git提供的示例脚本都带有.sample扩展名，是为了防止被意外运行。如果需要启用相应的钩子脚本，需要对其重命名（去掉.sample扩展名)。

> post-update
该钩子脚本由远程版本库的git receive-pack命令调用。当从本地版本库完成一个推送之后，即当所有引用都更新完毕后，在远程服务器上该钩子脚本被触发执行。

因此我们需要配置post-update钩子以便可以及时更新我们在VPS上存放Hexo 静态文件的目录。

```shell
# 回到hexo目录
cd /home/hexo
# 变成hexo用户
su hexo
# 使用hexo用户创建git裸仓库，以blog.git为例
git init --bare blog.git
# 进入钩子文件夹hooks
cd blog.git/hooks/
# 启用post-update
mv post-update.sample post-update
# 添加执行权限
chmod +x post-update
# 配置post-update
vim post-update
```
1. 注释如下行：

```shell
exec git update-server-info
```
2. 添加如下代码：

```shell
git --work-tree="静态文件VPS存放目录" --git-dir="刚才新建的VPS git地址" checkout -f
例：
git --work-tree=/home/hexo/blog --git-dir=/home/hexo/blog.git checkout -f
```

例：
![post-update](https://images.morethink.cn/d3f6cd5471afd285e9bb0599d1e8f8a3.png "post-update")


# Nginx

## 安装Nginx

```
yum install nginx
```

使用`nginx -v`查看，显示版本号则安装成功。

## Nginx配置

```
server {
        # 默认80端口
        listen       80 default_server;
        listen       [::]:80 default_server;
        # 修改server_name为自己之前注册好的域名，没有就不用更改
        server_name  morethink.cn;
        # 修改网站根目录，在这里存放你的Hexo静态文件，请自行选择或创建目录
        root         /home/hexo/blog;
        # 其他保持不变
        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
```

# 配置本地Hexo
找到本地Hexo博客的站点配置文件`_config.yml`，找到以下内容并修改：

```shell
deploy:
  type: git
  repo: hexo@你的服务器IP:/home/git/blog.git
  branch: master
```
然后在根目录执行以下命令：

```shell
hexo clean
hexo g -d
```

![大功告成](https://images.morethink.cn/d3fa2826d678f521535a4dce89bdf2f9.png "大功告成")



# 遇到的问题总结
1. 如果无法推送到vps，请检查hexo用户是否有权限操作所需目录
2. 关于` nginx root 403` 问题: 在我配置nginx碰到一个403问题，改了文件权限还是403，后来发现是nginx.conf中 user默认设置错了，把  `user nginx`改成`user root` 就好了。
3. deploy成功之后无法访问
    1. 查看vps静态目录是否有html文件，没有就是Git推送问题
    2. 查看Nginx配置是否成功


**参考文档：**
1. https://www.worldhello.net/gotgit/08-git-misc/070-hooks-and-templates.html

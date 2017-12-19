---
title: 怎么使用IDEA
date: 2017-5-7
tags: IDEA
categories: 工具
---
# war 和 war exploded
1. war部署首先通过IDEA生成.war工程文件，然后将WEB工程以包的形式上传到服务器，因此会替代服务器本来同名的web app项目。
2. war exploded模式直接将WEB工程以当前文件夹的位置关系上传到服务器。

<!-- more -->

使用：
1. war模式这种可以称之为是发布模式，看名字也知道，这是先打成war包，再发布；
2. war exploded模式是直接把文件夹、jsp页面 、classes等等移到IDEA生成的Tomcat部署文件夹里面，进行加载部署，不会替代本来tomcat中的同名web app项目。因此这种方式支持热部署，一般在开发的时候也是用这种方式。
3. 在平时开发的时候，使用热部署的话，应该对Tomcat中进行相应的设置,更改图中两处标记为 `update classes and resources`:
![](https://images.morethink.cn/IDEA-update.jpg)
这样的话修改的html，jsp或者第三方模版框架例如 Freemarker热部署才可以生效。

# 热部署插件 JRebel
> 在 Java Web 开发中， 一般更新了 Java 文件后要手动重启 Tomcat 服务器， 才能生效， 浪费不少生命啊， 自从有了 JRebel 这神器的出现， 不论是更新 class 类还是更新 Spring 配置文件都能做到立马生效，大大提高开发效率。

[IntelliJ IDEA 的 Java 热部署插件 JRebel 安装及使用](http://whudoc.qiniudn.com/2016/IntelliJ-IDEA-Tutorial/jrebel-setup.html)

# 常用插件

1. 阿里巴巴规约检测
2. 翻译插件

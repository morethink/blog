---
title: Git为某个域名设置代理
date: 2018-08-04
tags: Git
categories: 工具
---

国内访问Github很慢，可以通过配置代理来加快访问速度，但是对公司内部git服务器却不能使用代理。

下面通过更改Git配置文件对不同的域名使用不同的代理配置。

<!-- more -->

1. 打开Git 配置文件

    ```shell
    vi ~/.gitconfig
    ```
2. 添加如下配置：

    ```shell
    [http "https://github.com/"]
        proxy = http://127.0.0.1:1081
    [https "https://github.com/"]
        proxy = http://127.0.0.1:1081
    [http "https://my.comapnyserver.com/"]
        proxy = ""
    ```

---
title: Git同时push到多个远程仓库
date: 2018-04-03
tags: Git
categories: 工具
---

添加第二个远程地址时使用以下命令：
`git remote set-url --add origin git@github.com:morethink/programming.git`

查看远程分支：`git remote -v`

<!-- more -->

```
origin	git@git.coding.net:morethink/programming.git (fetch)
origin	git@git.coding.net:morethink/programming.git (push)
origin	hexo@MyHost2:/var/repo/gitbook.git (push)
```
也可以同时 push 到多个远程地址：`git push origin master`


```
Everything up-to-date
Everything up-to-date
```

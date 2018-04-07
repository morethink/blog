---
title: Git常用操作
date: 2018-04-03
tags: Git
categories: 工具
---

# Git同时push到多个远程仓库

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

# GitHub 更新已经 fork 的项目

1. clone 自己的 fork 分支到本地
可以直接使用 GitHub 客户端，clone 到本地，如果使用命令行，命令为：
`$ git clone  git@github.com:morethink/git-recipes.git`
2. 进入仓库，增加源分支地址到你项目远程分支列表中
此处是关键，先得将原来的仓库指定为 upstream，命令为：
`$ git remote add upstream git@github.com:geeeeeeeeek/git-recipes.git`
此处可使用 `git remote -v` 查看远程分支列表
    ```
    $ git remote -v
    origin	git@github.com:morethink/git-recipes.git (fetch)
    origin	git@github.com:morethink/git-recipes.git (push)
    upstream	git@github.com:geeeeeeeeek/git-recipes.git (fetch)
    upstream	git@github.com:geeeeeeeeek/git-recipes.git (push)
    ```
3. fetch 源分支的新版本到本地
`$ git fetch upstream`
4. 合并两个版本的代码
`$ git merge upstream/master`
5. 将合并后的代码 push 到 GitHub 上去
`$ git push origin master`

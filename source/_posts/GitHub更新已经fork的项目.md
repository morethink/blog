---
title: GitHub更新已经fork的项目
date: 2018-04-22
tags: Git
categories: 工具
---

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
4. 切换到本地master分支
`$ git checkout master`
5. 合并两个版本的代码
`$ git merge upstream/master`
6. 将合并后的代码 push 到 GitHub 上去
`$ git push origin master`

**参考文档：**
1. 添加远程分支
https://help.github.com/articles/configuring-a-remote-for-a-fork/
2. 完成同步
https://help.github.com/articles/syncing-a-fork/

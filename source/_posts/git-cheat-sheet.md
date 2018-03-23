---
title: 常用 git 命令备忘
date: 2018-03-23 11:02:43
tags: git
---

导出项目文件

`git archive --format zip -o filename.zip HEAD`

修剪远程分支

`git remote prune origin`

显示本地分支与远程分支跟踪关系

`git branch -vv`

重命名本地分支

`git branch -m oldname newname`

本地分支与远程分支建立关系

`git branch --set-upstream-to=origin/<branch> <cur branch>`

强制覆盖本地文件的修改

```
git fetch
git reset --hard origin/master 或
git reset --hard origin/develop 依此类推，使用想用的分支
```
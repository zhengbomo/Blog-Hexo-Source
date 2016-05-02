---
title: 删除远程分支
date: 2016-04-19 18:08:29
tags: [git]
categories: 技术
---


有时候会不消息把本地一些小分支push到远程服务器，删除远程分支与本地不一样，可以通过下面命令删除

<!-- more -->

删除本地分支

```git
$ git branch -D branch-name
```

解除远程分支的track关系（本地）

```git
$ git branch -r -d origin/branch-name
```

推送空到远程分支
```git
$ git push origin :branch-name
```

也可以使用
```git
$ git push origin --delete master
```

---
title: "git note"
date: 2018-08-29T19:55:29+08:00
categories: ["git"]
---

##### 获取新远程分支
从eclipse git远程时只选了一个远程分支之后，想要切换到新的远程分支在eclipse Git视图的Remote Tracking下面是找不到更多的远程分支的，需要对项目同步一下，即可看到，同时时可以看到其实就是执行的git fetch

##### 历史版本切换
在eclipse中右键team->shouw in history显示历史版本，选定版本右键Check out,恢复之前版本：项目右键->team->switch to master(当前分支即可)

##### 在eclipse无法pull
无法pull且大面积报红时，有可能是有冲突文件未提交导致不能pull,但大面报红又无法找到具体某一个时，切换至git视图pull,再次pull会提示某一个具体文件冲突

##### git reset --hard
HEAD引用指向给定提交，索引内容和工作目录内容都会变给定提交时的状态。也就是在给定提交后所修改的内容都会丢失(新文件会被删除，不在工作目录中的文件恢复，未清除回收站的前提)。
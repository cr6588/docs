---
title: "Git Cache"
date: 2018-04-20T13:50:56+08:00
draft: true
---

### git忽略已经被提交的文件
https://www.jianshu.com/p/2345b2aede59

git rm --cached xxx.xx *(所有)
在.gitignore中添加要忽略的文件
commit
push
其他成员pull，working directory中对应的文件会删除，所以如果文件重要，要提前备份。

---
title: "Note"
date: 2018-12-14T10:36:37+08:00
draft: true
---
查找1080端口pid
C:\Users\cr>netstat -aon|findstr "1080"
  TCP    127.0.0.1:1080         127.0.0.1:27015        ESTABLISHED     4304
  TCP    127.0.0.1:27015        127.0.0.1:1080         ESTABLISHED     1876
查找pid对应进程
C:\Users\cr>tasklist|findstr "4304"
iTunesHelper.exe              4304 Console                    1     14,016 K
---
title: "注册ip异常"
date: 2021-11-10T09:58:50+08:00
draft: false
---

启动的时候dubbo注册到一个未知的ip

````
  [DUBBO] No spring extension (bean) named:module, try to find an extension (bean) of type org.apache.dubbo.config.ModuleConfig, dubbo version: 2.7.5, current host: 192.168.199.202
````

搜索一番都让修改dns，于是尝试修改了一下dns
61.139.2.69
暂时好了
连接其他网络时有可能无法联网，后面又去除了。后面又加了一个本机hostname以及内外ip
````
192.168.199.202 cr6588.lan
````
重启又好了，然后又去除，再重启也好了。挺奇怪的。。。
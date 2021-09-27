---
title: "Scp Sftp Rysnc"
date: 2021-09-26T22:54:20+08:00
draft: true
---

# scp、sftp、rsync

三者都可以上传文件，其中scp上传文件时可直接上传不需要进行交互，sftp需要进行交互通过脚本上传时需要expect配合。rsync更加高级，它会通过算法计算出哪些文件有变化 ，然后只上传有变化的文件。下面分别进行实例说明

## scp

将erp-spring-boot下的文件上传到192.168.1.206的/home/chenyi/erp/，会被要求输入密码
scp erp-spring-boot/* root@192.168.1.206:/home/chenyi/erp/

当不想输入密码时有2中方式
1.使用ex


2.使用ssh密钥对

## sftp


> windows可使用bitvise软件的sftpc命令，详见bitvise.md
## rsync

    #将tt.png上传到192.168.1.206的/home/chenyi/erp/,-i输出所有更新的变更摘要
    rsync -i tt.png root@192.168.1.206:/home/chenyi/erp
    
    root@192.168.1.206's
    <f..T...... tt.png 无变化
    
    D:\test>rsync -i tt.
    root@192.168.1.206's
    <f+++++++++ tt.png 有变化
> windows有个[cwRsyn client](https://itefix.net/cwrsync)类似
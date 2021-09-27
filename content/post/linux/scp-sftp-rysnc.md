---
title: "Scp Sftp Rysnc"
date: 2021-09-26T22:54:20+08:00
draft: true
---

# scp、sftp、rsync

三者都可以上传文件，其中scp上传文件时可直接上传不需要进行交互，sftp需要进行交互，通过脚本上传时需要expect配合。rsync更加高级，它会通过算法计算出哪些文件有变化 ，然后只上传有变化的文件。下面分别进行实例说明

## scp

    #将erp-spring-boot下的文件上传到192.168.1.206的/home/chenyi/erp/，会被要求输入密码
    scp erp-spring-boot/* root@192.168.1.206:/home/chenyi/erp/

当不想输入密码时可使用ssh密钥对

    #前往当前用户的.ssh目录生成名为207的ssh密钥对
    cr6588@cr6588-2 .ssh % cd ~/.ssh
    cr6588@cr6588-2 .ssh % ssh-keygen -t rsa
    Generating public/private rsa key pair.
    Enter file in which to save the key (/Users/cr6588/.ssh/id_rsa): 207
    Enter passphrase (empty for no passphrase): 
    Enter same passphrase again: 
    Your identification has been saved in 207.
    Your public key has been saved in 207.pub.
    The key fingerprint is:
    SHA256:qN0Cb+TDXEcCDUSrYpRyO9hKSwjuzIx5bOsJvDnrbjY cr6588@cr6588-2.local
    The key's randomart image is:
    +---[RSA 3072]----+
    |     o=o         |
    |   .   o.        |
    |o +   . . .      |
    |+* . . . o       |
    |o+* o o S .      |
    |X=.o X o .       |
    |=B+ . X .        |
    | Eoo . o         |
    |=BB              |
    +----[SHA256]-----+
    #将207.pub上传到目标机器的authorized_keys文件中
    cr6588@cr6588-2 .ssh % scp 207.pub root@192.168.1.206:~/.ssh/authorized_keys
    root@192.168.1.206's password: 
    207.pub                                       100%  575    81.9KB/s   00:00   
    #由于有多个密钥对因此带上刚产生的参数，此时再次上传文件不需要密码.若生成ssh不输入名字时则不需要
    cr6588@cr6588-2 erp % scp  -i ~/.ssh/207 erp-spring-boot/* root@192.168.1.206:/home/chenyi/erp/
## sftp

    cr6588@cr6588-2 erp % sftp root@192.168.1.206
    root@192.168.1.206's password: 
    Connected to 192.168.1.206.
    sftp> put erp-spring-boot/* /home/chenyi/erp

当不想输入密码时可使用expect
> expect是一个自动化交互套件，主要应用于执行命令和程序时，系统以交互形式要求输入指定字符串，实现交互通信。
expect自动交互流程：
spawn启动指定进程---expect获取指定关键字---send向指定程序发送指定字符---执行完成退出。
注意该脚本能够执行的前提是安装了expect

    expect << EOF
     #expect 默认超时10s,-1永不超时
     set timeout -1
     spawn sftp root@192.168.1.206
     expect "ssword:" { send "密码\r" }
     expect "ftp> " { send "put erp-spring-boot/* /home/chenyi/erp\r" }
     expect "ftp> " { send "exit\r" }
    EOF

> windows可使用bitvise软件的sftpc命令，详见bitvise.md
## rsync

    #将tt.png上传到192.168.1.206的/home/chenyi/erp/,-i输出所有更新的变更摘要
    rsync -i tt.png root@192.168.1.206:/home/chenyi/erp
    
    root@192.168.1.206's
    <f..T...... tt.png 无变化
    
    D:\test>rsync -i tt.
    root@192.168.1.206's
    <f+++++++++ tt.png 有变化

免密码有2种方式

    # -e指定所要使用的远程shell程序,带上ssh
    #ssh key
    rsync -e "ssh -i ~/.ssh/207" -id erp-spring-boot/ root@192.168.1.206:/home/chenyi/erp/

    #expect
    #手动执行可带*，与expect结合且有exp_continue使用时*会提示目录不存在
    #spawn rsync erp-spring-boot/*
    #加上-d标识当前目录，不递归.加上-r表递归，-i输出所有更新的变更摘要
    expect << EOF
        #expect 默认超时10s,-1永不超时
        set timeout -1
        spawn rsync -ir erp-spring-boot/ root@192.168.1.206:/home/chenyi/erp/
        expect "ssword:" { send "密码\r";exp_continue }
    EOF

> windows有个[cwRsyn client](https://itefix.net/cwrsync)类似
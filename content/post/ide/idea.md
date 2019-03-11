---
title: "idea"
date: 2018-04-23T09:44:55+08:00
draft: true
---

##### 配置maven,jdk

##### 显示空格
File -> Settings -> Editor -> General -> Appearance -> Show whitespaces
1. 空格的点显示太小，如何居中或者更清晰？（字号调大仍然太小）
使用微软雅黑字体
File -> Settings ->  Appearance -> Override default fonts..打勾

2018idea中会默认清除多余空格
##### 序列化警告提示
idea implements Serializable接口后没有序列化id不会有警告提示
Setting->Inspections->Serialization issues->Serializable class without ’serialVersionUID’

##### 安装.ignore插件
File->Settings->Plugins->Browse repositories,搜索.ignore，点击Install.
在项目上右键->New ->.ignore file ->.gitignore file(Git) 

for循环，写出it后会有相应提示
it
Alt+Shift+Up/Down，上/下移一行

手动提示默认是crtl+空格，由于被windows输入法快捷键占用改成alt+/

手动提示时是区分大小写的，将case sensitive completion改为none

Ctrl+F12，类似eclipse+o显示方法

psvm(public static void main缩写) main方法

sout System.out.println
##### 修改静态资源不用重启
https://blog.csdn.net/Connie1451/article/details/80346981
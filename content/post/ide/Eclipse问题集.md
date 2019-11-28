---
title: "Eclipse问题集"
date: 2018-08-14T11:48:04+08:00
categories: ["eclipse"]
---

##### 提示resource is out of sync with the filesystem
在学习webpack复制项目时经常出现resource is out of sync with the filesystem.
![x](/images/ide/eclipse_res_out_filesys.png)
Right-click > Refresh will always clear this.

##### 在比较长的行末尾编辑时，窗口自动滚动到行首
这个问题很烦人，困扰很久终于解决
参考https://stackoverflow.com/questions/5353983/eclipse-after-editing-long-lines-editor-window-automatically-scrolls-to-beginni?newreg=c91d1937040d4b8594f0d9be357ba8e6

![x](/images/ide/x6qiz-o82mp.gif)

For java in eclipse :

Windows -> Preferences -> Java ->Editors ->Folding -> Enable Folding (uncheck)

For HTML, JSP, XML etc in eclispe : Windows -> Preferences -> General -> Editors -> Structured Text Editors -> Enable Folding (uncheck) (即取消折叠，关闭后在页面没有dom节点前的加减号)

![x](/images/ide/disable_Folding.png)
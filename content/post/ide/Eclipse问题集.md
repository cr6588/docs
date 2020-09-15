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

##### 打开js文件下载工具长时间无反应

打开了了一个vue相关的js文件，然后eclipse自动去下载了一个关联工具导致eclipse因为网络一直无法下载，强制关掉进程重开仍是这样，且不停增大内存。最后就让它下，最终提示因网络错误无法下载xxxx,之后重启一切正常
将该项目先转移它路径，防止下次打开仍去下载

##### 项目右键commit时，自动将文件add to index
在更新到photo时，commit项目时，将大量不必提交的文件add index,在处理冲突时，多提交了文件。因此想禁用此功能。
于是在git设置里查找一番，最终在preferences->Team->git->committing->Automatically stage selected resources on commit前面的勾去掉
![x](/images/ide/eclipse_git_committing.png)

##### maven web项目tomcat7:run启动由于webapp有空META-INF导致项目异常


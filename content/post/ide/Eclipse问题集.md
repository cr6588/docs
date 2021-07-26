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

##### eclipse_2021_06使用lombok一直提示Errors occurred during the build

编译项目一直提示

    Unable to make protected final java.lang.Class java.lang.ClassLoader.defineClass(java.lang.String,byte[],int,int) throws java.lang.ClassFormatError accessible: module java.base does not "opens java.lang" to unnamed module @645dc557

原因是jdk的兼容性问题导致
参照https://github.com/projectlombok/lombok/issues/2882

在eclipse.ini末尾增加
--illegal-access=warn
--add-opens java.base/java.lang=ALL-UNNAMED

##### eclipse 打开文件时有一条竖线

    参考自https://stackoverflow.com/questions/27410765/remove-vertical-line-in-eclipse-editor
    General -> Editors -> Text Editors -> Show Print Margin

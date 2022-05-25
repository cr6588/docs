---
title: "Vs"
date: 2020-05-19T10:38:53+08:00
draft: false
---

##### vs code 单击文件是预览模式，窗口上的文件名是斜体，打开其它文件是会覆盖显示。再双击文件名则会正体显示，新开文件也不会覆盖
相关设置是 "workbench.editor.enablePreview": true

##### 显示方法列表
Shift+Ctl+O

##### tomcat for vs
新增tomcat文件目录后，什么提示也没有。使用cleaner one pro彻底删除vscode重新安装tomcat for java插件
对web项目的target目录/项目目录使用debug tomcat，可直接热代码替换
访问直接使用根目录不需要前缀再配置文件的host下增加
<Context path="" docBase="增加到tomcat server目录名称" debug="0" reloadable="false"></Context>
直接右键停止有可能异常，无法中断。采用命令，7-12根据实际变更
ps -ef|grep apache-tomcat|grep -v grep|cut -c 7-12|xargs kill -9

##### 调试栏位置
默认启动debug后调试栏位置一致在上面会遮住一些信息显示，所以在设置->调试->Tool bar location->docker,改为停靠，只出现在调试视图中

##### 显示序列化

https://www.cnblogs.com/zhiyiYo/p/15318579.html

##### 手动提示
在vs code中这个快捷键有多个，在设置->键盘快捷方式->触发建议中可看到

当在实现类要自动产生未实现的方法时，也可使用此方式，再输入或选择接口

关闭智能代码提示
“editor.quickSuggestions”: false,
“editor.suggestOnTriggerCharacters”: false
##### 设置java import顺序
.vscode / settings.json：
{
  "java.completion.importOrder": ["java", "javax", "org", "com"]
}
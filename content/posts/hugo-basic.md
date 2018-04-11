---
title: "Hugo Basic"
date: 2018-04-10T19:17:21+08:00
draft: false
---

将hugo构建的内容部署至github-pages后，整个排版还是有很多问题，现在逐步完善菜单、分页的问题
#### 1.关于主题
最先使用了一个叫[code-editor](https://themes.gohugo.io/hugo-code-editor-theme/)的主题，按照官方文档中[Menu](https://gohugo.io/content-management/menus/)的介绍试了下，始终没成功。于是去主题中换成[docdock](https://themes.gohugo.io/docdock/)发现启动之后有一些提示，且在它git clone之后的exampleSite中有一些默认的配置，直接根据需要进行设置。所以最开始选择主题时最好选择在主题页面文档比较多或者在它的exampleSite中示例比较详细的主题进行学习。
在换成[docdock](https://themes.gohugo.io/docdock/)主题后，发现这个主题插入static里面的图片链接无法使用。正常插入图片代码

    ![x](xxx.png)

在[code-editor](https://themes.gohugo.io/hugo-code-editor-theme/)能成功显示，[docdock](https://themes.gohugo.io/docdock/)就不行了，有句mmp我一定......应该还是没搞懂hugo的一些基本结构。果断切换成另外一个主题[potato-dark](https://themes.gohugo.io/potato-dark/)在看了它里面的exampleSite代码后发现它的图片路径加了个/且它用了一个jquery插件，果断又去改成
```
![x](/xxx.png)
```
图片终于正常显示，果断感觉自己2了。

#### 关于菜单
在换成[docdock](https://themes.gohugo.io/docdock/)主题。发现页面这个提示
![x](/import_docs.png)
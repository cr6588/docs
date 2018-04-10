---
title: "Hugo Github Pages"
date: 2018-04-10T11:48:57+08:00
draft: true
---

将hugo生成的静态网页部署至Github Pages

### 1.参见[Hugo](http://gohugo.io/)官方文档进行安装 
### 2.完成Quick Start
### 3.构建部署至Github Pages

若hugo产生的站点在D:\git\docs目录，进入docs目录

编辑config.toml,cr6588替换成你github的用户名
```
baseURL = "https://cr6588.github.io/"
languageCode = "en-us"
title = "My New Hugo Site"
theme = "ananke"
```
构建（-D, include content marked as draft）
```
hugo -D
```
顺利的话会产生public目录里面的文件就是最终需要发布到Github Pages的内容

在github创建cr6588.github.io(将cr6588替换成你的用户名)
```
cd public
git init
git remote add origin https://github.com/cr6588/cr6588.github.io.git
git add -A
git commit -m "first commit"
git push -u origin master
```
访问https://cr6588.github.io/ 即可看到效果
### 4.将源码发布至github
cr6588.github.io已经发布，但docs这个站点我也想发布至git保存，于是类似在github新建一个docs，然后在docs目录
```
git remote add origin https://github.com/cr6588/docs.git
git add -A
git commit -m "first commit"
git push -u origin master
```
然后在你的ide或者直接cmd从github上面clone到另外一个目录，导入完成之后会发现缺少public,themes目录，在此目录使用hugo server -D会提示缺少主题目录文件。这里引入一个git子模块概念(git submodule),普通导入是没有将git submodule引入的。此时输入git submodule init提示
```
fatal: No url found for submodule path 'public' in .gitmodules
```
在.gitmodules文件没有发现public说明，是因为之前在docs/public目录时并没有将其作为子模块加入到docs中，所以此时在.gitmodules手动添加
```
[submodule "public"]
    path = public
    url = https://github.com/cr6588/cr6588.github.io.git
```
再次使用
```
git submodule init
git submodule update
```
等待将相关子模块依赖下载完成。吐槽一下国内github速度太慢了......

然后hugo server -D测试一下,没有问题将修改后的.gitmodules提交.再次导入时仍然需要将子模块依赖导入

### Tips
若访问空白使用hugo -D查看构建是否有错误原因，我再次重建使用时发现主题有点问题，换了个主题
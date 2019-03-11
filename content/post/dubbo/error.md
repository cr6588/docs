---
title: "Error"
date: 2019-02-11T13:58:57+08:00
draft: true
---

1.解压dubbojar包，拿到dubbo.xsd文件

2.在eclipse中Windows->Preferences->XML->XML Catalog -> Add->Catalog Entry  ->File System 选择刚刚的文件路径

3.修改Key值为http://code.alibabatech.com/schema/dubbo/dubbo.xsd（与配置文件中相同），然后保存

4.在xml文件右键点击Validate即可解决问题。
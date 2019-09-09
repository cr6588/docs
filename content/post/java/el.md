---
title: "El"
date: 2019-08-13T10:35:40+08:00
draft: true
---

js中使用el表达式，在null判断时若只有'',其结果如图
![x](/images/el1.png)

![x](/images/el2.png)

若只有"",其结果如图

![x](/images/el2.png)

所以最好使用
 var platAcceptJSON = ${platAcceptJSON == null ? '""' : platAcceptJSON};
 或者
 var platAcceptJSON = ${platAcceptJSON == null ? "''" : platAcceptJSON};
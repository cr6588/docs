---
title: "DeleteImages"
date: 2019-10-21T13:48:25+08:00
draft: false
---

# 定时删除多余镜像

发版次数较多之后本地会产生很多镜像。之前都是人工删除，很是麻烦，所以写了一个定时任务删除来清除多余镜像

先写一个基础脚本deleteImages.sh.由于我的镜像都是以日期时间为版本，所以排序直接以时间降序

````shell
#!/bin/bash
name=$1
docker images|grep $name|awk '{print $1,$2}'|sort -t ' ' -k 2 -r|awk 'NR > 3 {printf "%s:%s ",$1,$2}'|xargs docker image rm
````
并加入权限chmod +x deleteImages.sh

其过程是用grep匹配name的image,之后用awk只要第1，2列，也就是image的名称与版本。然后用sort排序，-t ` `以空格分隔, -k 2以第2列即版本排序 -r是降序,接着再用awk只要大于第3行之后的行，且用printf(不是print)格式化输出符合镜像删除的格式（镜像名:版本），最后作为参数传给docker image rm即可。

在实际应用中有可能传递的name会不止匹配到某一个模块，例如log有机会匹配到log，也会匹配到含有log的名称。所以有时需要进行一些排除，将上面的脚本再加一个排除参数，很容易想到使用grep -v排除，deleteImages.sh修改后如下

````shell
#!/bin/bash
name=$1
#将name以版本降序排列，删除第3个以后的版本
#$2用于排除镜像
if [ x$2 != x ]; then
    docker images|grep $name|grep -v $2 |awk '{print $1,$2}'|sort -t ' ' -k 2 -r|awk 'NR > 3 {printf "%s:%s ",$1,$2}'|xargs docker image rm
else
    docker images|grep $name|awk '{print $1,$2}'|sort -t ' ' -k 2 -r|awk 'NR > 3 {printf "%s:%s ",$1,$2}'|xargs docker image rm
fi

````

若有多个模块镜像需要删除，则另写一个脚本deleteErpImages，在此脚本指定调用deleteImages.sh即可，类似如下
````shell
#!/bin/bash
./deleteImages.sh erp/erp-admin
./deleteImages.sh erp/erp-logistics
#erp-log删除时排除logistics
./deleteImages.sh erp/erp-log logistics
....
````

最后在crontab加入定时任务
crontab -e
* * */1 * * ./deleteErpImages.sh
保存即可
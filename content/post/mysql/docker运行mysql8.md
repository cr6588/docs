---
title: "docker运行mysql8"
date: 2019-02-11T15:34:04+08:00
categories: ["mysql"]
---

````bash
docker run -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD=密码 -d mysql:8
#由于密码加密方式与以前不同，所以需要进入容器更改密码加密方式
docker exec -it mysql /bin/bash
mysql -u root -p
输入密码
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'tTdAdf212';
不能启动时查看数据挂载磁盘信息，通过portainer查看宿主机磁盘位置，然后新开容器将磁盘数据复制到新容器的数据盘位置

#加载本地磁盘与自定义配置信息，使用主机网络
docker run -v /docker/mysql/custom:/etc/mysql/conf.d -v /docker/mysql/data:/var/lib/mysql --net=host --name mysql  -e MYSQL_ROOT_PASSWORD=密码 -d mysql:5.7.25

````

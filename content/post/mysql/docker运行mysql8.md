---
title: "docker运行mysql8"
date: 2019-02-11T15:34:04+08:00
categories: ["mysql"]
---

````bash
docker run -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD=密码 -d mysql:8
#由于密码加密方式与以前不同，所以需要进入容器更改密码加密方式
docker exec -it mysql /bin/bash
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '密码';
````

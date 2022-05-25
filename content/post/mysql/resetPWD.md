---
title: "mysql重置密码"
date: 2021-02-07T11:17:17+08:00
draft: false
---

参照 https://blog.csdn.net/weixin_36511527/article/details/87865212

1、在[mysqld]中添加

````
#跳过权限
skip-grant-tables
````
2、重启mysql服务

service mysqld restart
3、用户登录

mysql -uroot -p (直接点击回车，密码为空)
选择数据库

use mysql;
下面我们就要修改密码了

以前的版本我们用的是以下修改

update user set password=password('root') where user='root';
但是在5.7版本中不存在password字段，所有我们要用以下修改进行重置密码

update user set authentication_string=password('123456') where user='root';
执行

flush privileges;
4、退出mysql

quit;
5、将最开始修改的配置文件my.cnf中的skip-grant-tables删除

6、重启mysql

7、当你登陆mysql之后你会发现，当你执行命令时会出现

ERROR 1820 (HY000): You must reset your password using ALTER USER statement；
这是提示你需要修改密码

当你执行了

SET PASSWORD = PASSWORD('123456');
如果执行成功后面的就不要看了，纯属浪费时间！

如果出现：

ERROR 1819 (HY000): Your password does not satisfy the current policy requirements
你需要执行两个参数来把mysql默认的密码强度的取消了才行

set global validate_password_policy=0; set global validate_password_mixed_case_count=2;
这时你再执行

SET PASSWORD = PASSWORD('123456');
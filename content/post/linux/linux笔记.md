---
title: "linux笔记"
date: 2018-04-17T20:56:48+08:00
categories: ["linux"]
---

//centos6
//显示linux发行版本信息
cat /etc/issue
cat /etc/os-release
//查看内存
free -m
//防火墙
vi /etc/sysconfig/iptables
service iptables status
service iptables restart
//加权限
chmod u+rwx filename
//后台运行
nohup java -jar erp-log.jar > base-service.log 2>&1 &
//vi复制行
1）把光标移动到要复制的行上
2）按yy
3）把光标移动到要复制的位置
4）按p

#返回上一次目录
cd -
#!/bin/bash
nohup java -jar erp-cache.jar > erp-cache.log 2>&1 &

//linux安装字体
Linux字体文件放在/usr/share/font/，只要将字体文件拷贝到这里就可以了。
这里示例安装Windows的所有字体。
    1，新建文件夹，装Windows字体，方便管理。打开终端输入命令:mkdir win。
    2，复制Windows下 的所有字体。cd命令切换到C盘挂载的目录，进入c:\windows\Fonts。使用cp命令复制字体：cp *.ttf *.TTF /home/username/win/。这样就把Windows的字体复制到了主目录下的win目录里面。
    3，安装字体。终端输入:mv /home/username/win/ /usr/share/fonts/。移动到Linux字体库中。
    4，刷新系统即刻生效，输入命令：sudo fc-cache -fv。
//将jar包安装到本地repository中

mvn install:install-file -Dfile=my-jar.jar -DgroupId=org.richard -DartifactId=my-jar -Dversion=1.0 -Dpackaging=jar
mvn install:install-file -Dfile=ocean.client-1.0.jar -DgroupId=com.alibaba -DartifactId=ocean.client -Dversion=1.0 -Dpackaging=jar

1、*.tar 用 tar –xvf 解压
2、*.gz 用 gzip -d或者gunzip 解压
3、*.tar.gz和*.tgz 用 tar -xzf 解压
4、*.bz2 用 bzip2 -d或者用bunzip2 解压
5、*.tar.bz2用tar –xjf 解压
6、*.Z 用 uncompress 解压
7、*.tar.Z 用tar –xZf 解压
8、*.rar 用 unrar e解压
9、*.zip 用 unzip 解压

tar -cvf etcbak.tar etc/          打包一个tar

tar -xvf etcbak.tar                 解开一个tar

 tar -xvf  etcbak.tar  -C ./dir  解开一个tar到当前的 dir 目录中
_______________________________________________________
tar -zcvf etcbak.tar.gz etc/ 打包压缩一个 tar
 
tar -zxvf etcbak.tar.gz  解压一个tar
_______________________________________________________
 
--B
zip -r mybatis.zip    ./mybatis 将目录./mybatis压缩成一个mybatis.zip 文件
unzip  mybatis.zip  -d ./newdir 将mybatis.zip 文件解压缩到目录  ./newdir
 
_______________________________________________________
 
--C gzip 只能压缩文件 
gzip  file.txt   --压缩文件 
gunzip file.txt.gz  --解压缩文件 


mv 源文件 目标文件
//强制递归复制文件
cp -rf erp-web-tomcat-7.0.82_9001_9002_9003 erp-admin-tomcat-7.0.82_9004_9005_9006
#不提示覆盖
\cp -rf t tt


phantomjs用于保存标签图片
1.下载phantomjs
    linux:wget https://bitbucket.org/ariya/phantomjs/downloads/phantomjs-2.1.1-linux-x86_64.tar.bz2(linux下载后文件名过长用mv 更改前名称 phantomjs-2.1.1-linux-x86_64.tar.bz2)
    windows:https://bitbucket.org/ariya/phantomjs/downloads/phantomjs-2.1.1-windows.zip
2.解压
    linux:tar -jxvf phantomjs-2.1.1-linux-x86_64.tar.bz2
3.加入环境变量
    linux:
        vi /etc/profile
        加入export PATH=/home/phantomjs-2.1.1-linux-x86_64/bin:$PATH
        使环境变量生效,在当前窗口以及以后新建立的ssh生效，以前打开的窗口不会生效
        source /etc/profile
        查看版本是否生效，若出现phantomjs: error while loading shared libraries: libfontconfig.so.1: cannot open shared object file: No such file or directory，安装（yun install xxx）相应依赖
        phantomjs --version
4.linux加入字体
    1.复制font.zip到linux的/usr/share/fonts/目录下
    2.unzip font.zip
    3.刷新系统即刻生效，输入命令：fc-cache -fv
    
    
host文件位置：/etc/hosts
vi /etc/hosts即可编辑
修改方式类似windows.


Linux centos重启命令：

　　1、reboot
　　2、shutdown -r now 立刻重启(root用户使用)
　　3、shutdown -r 10 过10分钟自动重启(root用户使用)
　　4、shutdown -r 20:35 在时间为20:35时候重启(root用户使用)
　　如果是通过shutdown命令设置重启的话，可以用shutdown -c命令取消重启
　Linux centos关机命令：
　　1、halt 立刻关机
　　2、poweroff 立刻关机
　　3、shutdown -h now 立刻关机(root用户使用)
　　4、shutdown -h 10 10分钟后自动关机
linux下查看当前目录属于哪个分区？
    -h 单位g
    df -h /opt/test
linux安装redis 4.0
下载解压
make
cd src
./redis-server
停止
./redis-cli -h 127.0.0.1 -p 6379 shutdown
时间同步
授时服务器采用阿里云 https://help.aliyun.com/knowledge_detail/40583.html

ntpdate -u ntp1.aliyun.com
定时同步服务器时间
crontab -e
*/10 * * * * /usr/sbin/ntpdate ntp1.aliyun.com

文件的属主和属组属性设置

chown user:market f01　　//把文件f01给uesr，添加到market组
ll -d f1  查看目录f1的属性

javac *.java会生成对应的*.class文件
java *.class就可以执行了，.class可以省略
找不到主类，加入classpath=jdk/lib && export PATH=$classpath:PATH
移除临时环境变量unset erp_version

查看目录文件大小
du -sh *

批量当前以及子目录清空日志文件内容
for i in `find . -name "*.log"`; do cat /dev/null >$i; done

将curl的请求结果输出值aaaa.txt文件中。``执行命令,不是单引号''，-i打印请求头，-H添加请求头。多个时在-h xxx, -d数据请求 @jenkins-token.txt当前目录下的jenkins-token.txt文件，不用文件是不添加@。-X请求方式
echo `curl -i -H "Content-Type:application/json" -d @jenkins-token.txt -X POST https://iam.myhuaweicloud.com/v3/auth/tokens` > aaaa.txt

清除从window传到linux的^M符号,g全局，^M实际是crtl+v,crtl+m,输入时也需要crtl+v,crtl+m，不能直接复制^M
sed -i 's/^M//g' ELB.sh

对于grep贪婪匹配，需使用perl语法
即grep -P "weight\": .*?,"

(Linux查看物理CPU个数、核数、逻辑CPU个数)[https://www.cnblogs.com/emanlee/p/3587571.html]
````shell
# 总核数 = 物理CPU个数 X 每颗物理CPU的核数 
# 总逻辑CPU数 = 物理CPU个数 X 每颗物理CPU的核数 X 超线程数

# 查看物理CPU个数
cat /proc/cpuinfo| grep "physical id"| sort| uniq| wc -l

# 查看每个物理CPU中core的个数(即核数)
cat /proc/cpuinfo| grep "cpu cores"| uniq

# 查看逻辑CPU的个数
cat /proc/cpuinfo| grep "processor"| wc -l
````

#ip 查询
curl cip.cc

#tcp 连接查询
yum install net-tools
netstat -na

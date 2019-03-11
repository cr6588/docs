通过virtualbox安装CentOS-7-x86_64-Minimal-1804.iso后网络模式选择桥接重启
#禁用防火墙
systemctl stop firewalld
#yum install net-tools
ifconfig
#远程连接后参照 [Centos官方安装文档](https://docs.docker.com/install/linux/docker-ce/centos/#install-docker-ce-1)

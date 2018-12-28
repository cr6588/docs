---
title: "Redis"
date: 2018-05-31T14:38:33+08:00
draft: true
---

# docker redis 宿主机不能访问
按照https://redis.io/download 下载并编译redis后
构建redis镜像，开放6379端口并设置密码，在容器内使用./redis-cli能够访问，在宿主机一直不能连接。同样方式开放zookeeper的2181端口并无问题。排除端口映射的问题，网上搜索根据https://blog.csdn.net/myminner/article/details/79060264 中指出了在默认的redis.conf中bind只监听127.0.0.1，在其后加入容器ip即可.之后直接使用./redis-server &启动未带配置文件参数。在宿主机访问发现提示

    (error) DENIED Redis is running in protected mode because protected mode is enabled, no bind address was specified, no authentication password is requested to clients. In this mode connections are only accepted from the loopback interface. If you want to connect from external computers to Redis you may adopt one of the following solutions: 1) Just disable protected mode sending the command 'CONFIG SET protected-mode no' from the loopback interface by connecting to Redis from the same host the server is running, however MAKE SURE Redis is not publicly accessible from internet if you do so. Use CONFIG REWRITE to make this change permanent. 2) Alternatively you can just disable the protected mode by editing the Redis configuration file, and setting the protected mode option to 'no', and then restarting the server. 3) If you started the server manually just for testing, restart it with the '--protected-mode no' option. 4) Setup a bind address or an authentication password. NOTE: You only need to do one of the above things in order for the server to start accepting connections from the outside.

再参考bind官方说明

    # By default, if no "bind" configuration directive is specified, Redis listens
    # for connections from all the network interfaces available on the server.
    # It is possible to listen to just one or multiple selected interfaces using
    # the "bind" configuration directive, followed by one or more IP addresses.

提示说明默认是启用了保护模式的，所以最后去除了redis.conf中的bind设置（宿主机做好防火墙配置），但是保留了requirepass这样宿主机即可正常通过./redis-cli链接之后，auth 密码即可
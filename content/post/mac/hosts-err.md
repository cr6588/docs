---
title: "记录一次复制hosts引发的问题"
date: 2021-12-27T15:33:36+08:00
draft: false
---

## 过程
由于更换电脑，当把以前windows的hosts文件复制到新电脑后。启动项目连接zookeeper时提示超时。
在网上一番搜索后，走上了歧路。。。

都提示是有可能超时时间不够，于是加上了超时时间?timeout=600000
````
<dubbo:registry protocol="zookeeper" address="zookeeper.xxx.cn:2181?timeout=600000" />
````
在这之后到是可以连接，只是每次都很慢，有10多秒都是在注册zk。但在windows或者linux上没有这个问题，就这样带着疑问暂时搁置一段时间。
最近有空加上实在忍不住了，来看看到底是哪个地方执行很慢。

首先能够确定是连接zk比较慢，于是找了一个连接zk的demo代码，类似这种
````
       BaseZookeeper zookeeper = new BaseZookeeper();
       zookeeper.connectZookeeper("192.168.1.206:2181");
````
然后跟踪connectZookeeper的代码，发现都是多线程的方法不好跟踪。遂查看日志发现每次都是在
````
2021-12-27 15:53:56 - org.apache.zookeeper.ClientCnxn$SendThread.startConnect(ClientCnxn.java:1058) - Opening socket connection to server /192.168.1.206:2181
````
此处停住进行等待，点开此处代码ClientCnxn.java:1058
````
            LOG.info("Opening socket connection to server " + addr);
            SocketChannel sock;
            sock = SocketChannel.open();
            sock.configureBlocking(false);
            sock.socket().setSoLinger(false, -1);
            sock.socket().setTcpNoDelay(true);
            setName(getName().replaceAll("\\(.*\\)",
                    "(" + addr.getHostName() + ":" + addr.getPort() + ")"));
````
之后发现addr.getHostName()非常慢，一路点进去在
InetAddress.class 676
InetAddress[] arr = InetAddress.getAllByName0(host, check);
````

                /* now get all the IP addresses for this hostname,
                 * and make sure one of them matches the original IP
                 * address. We do this to try and prevent spoofing.
                 */

                InetAddress[] arr = InetAddress.getAllByName0(host, check);
                boolean ok = false;

                if(arr != null) {
                    for(int i = 0; !ok && i < arr.length; i++) {
                        ok = addr.equals(arr[i]);
                    }
                }
````
此行执行较久。更里面的代码不怎么看得懂了，只是用ipv6做了些事。
此后写一个更直接的类似ClientCnxn.java:1058，获取hostname的方法
````
InetSocketAddress t = new InetSocketAddress("192.168.1.206", 2181);
System.out.println(t.getHostName());
````
发现依然很慢才能获取。但在windows上执行很快。
此时我将192.168.1.206换成另外一个192.168.1.171
发现连接非常快。对比2个服务器的zk配置后,发现没什么不同。
那么还是自己这台电脑的问题。
192.168.1.206在本机是做了一个host的，192.168.1.171则没有。
于是我清空hosts发现192.168.1.206也连接很快了。
此时就清晰了，就是hosts文件的问题。由于用了一个第三方的软件来更改hosts文件
始终没发现什么问题，在试了很多次之后用vi /etc/hosts看了一下有问题的hosts文件
````

        192.168.1.206       zookeeper.xxx.cn^M
        192.168.1.206 user.xxx.xxx.cn

~              
````

问题一下就暴露了，最后多了^M。
## 原因
是由于windows与linux/unix的换行符不同导致的。
用vi手动编辑之后一切就好了。
## 总结
之前其实遇到过类似换行符问题，但这次最后确实没想到是这个
最后总结拷贝配置文件时，一定注意换行符的问题

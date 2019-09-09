---
title: "Openvpn"
date: 2019-08-05T10:43:15+08:00
draft: true
---

# [install server](https://www.digitalocean.com/community/tutorials/how-to-set-up-and-configure-an-openvpn-server-on-centos-7)

# install client

````shell
yum install epel-release -y
yum install -y openvpn
cp ca.crt client.conf  client.crt  client.key  myvpn.tlsauth /etc/openvpn
systemctl -f enable openvpn@client.service
systemctl start openvpn@client
````

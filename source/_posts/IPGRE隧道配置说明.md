---
title: IP/GRE隧道配置说明
date: 2019-03-15 09:30:21
categories:
 - linux
tags:
 - IP/GRE
---


主要记录下IP/GRE隧道的配置说明。<!--more-->

举个例子，例如我们有两个阿里云账号，并且在不同vpc下，每个账号下各有一台云服务器ECS，IP/GRE隧道就是其中的一种方式,达到内网互通的目的，网络拓扑图如下：

<img alt="GRE隧道拓扑" height="400" src="https://dictator.oss-cn-hongkong.aliyuncs.com/hexo/title/ipgre.png">

> 注意：搭建隧道时需要在双方vpc下 添加路由表，安全组开放公网ip对端地址和内网对端ip地址（不限端口 -1/-1）


```bash
serverA 公网ip 30.11.177.121  内网ip 192.168.80.150

serverB 公网ip 30.11.177.86   内网ip 192.168.136.128
```

公网ip可ping通 ，内网ip互相ping不通，所以要建立个隧道 保证内网互通，这样可以节省成本，不用阿里云vpn网关 智能接入网关等产品。下面为配置命令，

serverA：

```bash
modprobe ip_gre  # 加载gre模块

ip tunnel add tun0 mode gre remote 30.11.177.86 local 30.11.177.121 ttl 255

ip link set tun0 up # 启动tun0

ip link set tun0 up mtu 1500

ip addr add 192.168.1.1 peer 192.168.1.2 dev tun0  #为tun0添加地址和对端地址

ip route add 192.168.136.0/24 via 192.168.1.2  #添加路由网关

```

serverB：


```bash
modprobe ip_gre

ip tunnel add tun0 mode gre local 30.11.177.86 remote 30.11.177.121 ttl 255

ip link set tun0 up

ip link set tun0 up mtu 1500

ip addr add 192.168.1.2 peer 192.168.1.1 dev tun0

ip route add 192.168.80.0/24 via 192.168.1.1

```

总结：上面此方法优点节省成本，配置上比搭建openvpn，openswan之类的较为简单，但是openswan等vpn运维起来较为麻烦。缺点是安全性有待考究，没有设置加密验证传输.
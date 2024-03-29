---
title: "容器网络和iptables"
linkTitle: "容器网络和iptables"
weight: 250
description: >
  容器网络和iptables
---

{{% pageinfo %}}
容器网络和iptables
{{% /pageinfo %}}




启动一个容器，查看iptables
```
$ iptables-save
# Generated by iptables-save v1.8.4 on Thu Jan 12 10:28:13 2023
*security
:INPUT ACCEPT [3207420:579585381]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [3205687:578296425]
COMMIT
# Completed on Thu Jan 12 10:28:13 2023
# Generated by iptables-save v1.8.4 on Thu Jan 12 10:28:13 2023
*raw
:PREROUTING ACCEPT [3281739:594537318]
:OUTPUT ACCEPT [3205687:578296425]
COMMIT
# Completed on Thu Jan 12 10:28:13 2023
# Generated by iptables-save v1.8.4 on Thu Jan 12 10:28:13 2023
*nat
:PREROUTING ACCEPT [5948:1205704]
:INPUT ACCEPT [120:27480]
:OUTPUT ACCEPT [21:1239]
:POSTROUTING ACCEPT [21:1239]
:DOCKER - [0:0]
-A PREROUTING -m addrtype --dst-type LOCAL -j DOCKER
-A OUTPUT ! -d 127.0.0.0/8 -m addrtype --dst-type LOCAL -j DOCKER
-A POSTROUTING -s 172.17.0.0/16 ! -o docker0 -j MASQUERADE
-A DOCKER -i docker0 -j RETURN
COMMIT
# Completed on Thu Jan 12 10:28:13 2023
# Generated by iptables-save v1.8.4 on Thu Jan 12 10:28:13 2023
*mangle
:PREROUTING ACCEPT [3281739:594537318]
:INPUT ACCEPT [3207420:579585381]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [3205687:578296425]
:POSTROUTING ACCEPT [3205687:578296425]
COMMIT
# Completed on Thu Jan 12 10:28:13 2023
# Generated by iptables-save v1.8.4 on Thu Jan 12 10:28:13 2023
*filter
:INPUT ACCEPT [187:49960]
:FORWARD DROP [0:0]
:OUTPUT ACCEPT [68:7214]
:DOCKER - [0:0]
:DOCKER-ISOLATION-STAGE-1 - [0:0]
:DOCKER-ISOLATION-STAGE-2 - [0:0]
:DOCKER-USER - [0:0]
-A FORWARD -j DOCKER-USER
-A FORWARD -j DOCKER-ISOLATION-STAGE-1
-A FORWARD -o docker0 -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A FORWARD -o docker0 -j DOCKER
-A FORWARD -i docker0 ! -o docker0 -j ACCEPT
-A FORWARD -i docker0 -o docker0 -j ACCEPT
-A DOCKER-ISOLATION-STAGE-1 -i docker0 ! -o docker0 -j DOCKER-ISOLATION-STAGE-2
-A DOCKER-ISOLATION-STAGE-1 -j RETURN
-A DOCKER-ISOLATION-STAGE-2 -o docker0 -j DROP
-A DOCKER-ISOLATION-STAGE-2 -j RETURN
-A DOCKER-USER -j RETURN
COMMIT
# Completed on Thu Jan 12 10:28:13 2023
```

## iptables 基础

一个 iptables 上可能包含多个表，表可能包含多个链。链可以是内置的或者用户定义的。
链可能包含多个规则，规则是为数据包定义的。

表是一堆链，链是一堆防火墙规则。

> iptables -> tables -> Chains -> Rules


## Iptables 表和链
四个内置表

### 1. 过滤表
默认表，表内有内置链。
- INPUT链 - 传入防火墙
- OUTPUT链 - 从防火墙传出
- FORWARD链 - 本地服务器上另一个NIC的数据包。对于通过本地服务器路由的数据包。

### 2. NAT表
NAT表具有以下内置链
- PREROUTING 链 - 在路由之前更改数据包。 用于 DNAT (目标NAT)
- POSTROUTING 链 - 在路由后更改数据包。用于SNAT（源 NAT）
- OUTPUT 链 - 防火墙上本地生成的数据包的NAT。

### 3. Mangle表
用于专门的数据包更改，改变TCP报文头中的QOS位，具有以下内置链：
- 

## Docker 网络与 iptables
dockerd daemon 参数有 --iptables 参数，默认为 true ，默认添加 iptables 规则。如果启动时添加 **--iptables=false** 参数，默认没有任何规则。

利用 docker in docker 启动一个镜像 docker:dind 的容器，进入之后查看 iptable 规则。

```
# Generated by iptables-save v1.8.6 on Thu Jan 12 02:35:29 2023
*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
:DOCKER - [0:0]
:DOCKER-ISOLATION-STAGE-1 - [0:0]
:DOCKER-ISOLATION-STAGE-2 - [0:0]
:DOCKER-USER - [0:0]
-A FORWARD -j DOCKER-USER
-A FORWARD -j DOCKER-ISOLATION-STAGE-1
-A FORWARD -o docker0 -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A FORWARD -o docker0 -j DOCKER
-A FORWARD -i docker0 ! -o docker0 -j ACCEPT
-A FORWARD -i docker0 -o docker0 -j ACCEPT
-A DOCKER-ISOLATION-STAGE-1 -i docker0 ! -o docker0 -j DOCKER-ISOLATION-STAGE-2
-A DOCKER-ISOLATION-STAGE-1 -j RETURN
-A DOCKER-ISOLATION-STAGE-2 -o docker0 -j DROP
-A DOCKER-ISOLATION-STAGE-2 -j RETURN
-A DOCKER-USER -j RETURN
COMMIT
# Completed on Thu Jan 12 02:35:29 2023
# Generated by iptables-save v1.8.6 on Thu Jan 12 02:35:29 2023
*nat
:PREROUTING ACCEPT [0:0]
:INPUT ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
:POSTROUTING ACCEPT [0:0]
:DOCKER - [0:0]
-A PREROUTING -m addrtype --dst-type LOCAL -j DOCKER
-A OUTPUT ! -d 127.0.0.0/8 -m addrtype --dst-type LOCAL -j DOCKER
-A POSTROUTING -s 172.18.0.0/16 ! -o docker0 -j MASQUERADE
-A DOCKER -i docker0 -j RETURN
COMMIT
# Completed on Thu Jan 12 02:35:29 2023
```

### DOCKER-USER 链

```
-A FORWARD -j DOCKER-USER
...
-A DOCKER-USER -j RETURN
```
以上规则是在 filter 表中生效的：
- 第一条表示流量进入 FORWARD 链后，直接进入到 DOCKER-USER 链
- 最后一条表示流量进入 DOCKER-USER 链处理后，可以再 RETURN 回原先的链，进行后续规则的匹配。

这个其实是 Docker 预留的一个链，供用户自行配置的一些额外的规则。


Docker 默认的路由规则是允许所有客户端访问的，如果 Docker 在公网上运行，或者希望避免 Docker 中的容器被局域网中其他客户端访问，那么需要在这里添加一条规则。

> iptables -I DOCKER-USER -i <net interface> ! -s xxx.xxx.xxx.xxx -j DROP （仅允许 xxx.xxx.xxx.xxx 访问）


Docker 在重启之类的操作时，会进行 iptables 相关规则的清理和重建，但是 DOCKER-USER 链中的规则可以持久化，不受影响。


### DOCKER-ISOLATION-STAGE-1/2 链
这两条链主要是分两个阶段进行了桥接网络隔离。

相同 network 的容器是可以 ping 通的，不同 network 的容器则不能。

### Docker 链
使用最频繁，规则最多。如果误删链中内容，可能会导致容器的网络出现问题，尝试手动修改和重启Docker。

Docker 分别在 filter 和 nat 表增加了规则，具体含义如下：

filter 表中新增容器的目标地址，目标端口是映射端口的 TCP 协议被接收。

nat 表 上提供了 Docker 容器端口转发的能力，将访问本地地址端口流量的目标地址换成容器的 ip:port 地址。

## containerd 与 iptables
k8s1.24不支持 dockershim，要将容器的运行时切换到 contained，在 contained 中是无法进行端口映射的。
自带的 ctr 命令不支持端口映射，那么想用的时候，怎么做呢？

一种是自己管理 iptables 规则，比较繁琐。
另一种方式是使用 nerdctl 这个专门为 containerd 做的，兼容 Docker CLI 的工具。

> nerdctl run  0d --name redis -p 6379:6379 redis:alpine



---
title: "基础知识"
linkTitle: "基础概念"
weight: 1
description: >
  网络基础概念
---

{{% pageinfo %}}
网络基础知识
{{% /pageinfo %}}

## vxlan
全称 Virtual extensible local area network，是用三层协议封装的二层协议，允许第二层数据包在第三层网络上传输。通过 Overlay 技术旨在扩展 vlan，
以解决大型云计算部署中虚拟网络数量不足的问题。

## DSR（Direct Server Return）
默认情况下，cilium 的 BPF NodePort 实现以 SNAT 模式运行。也就是说，当节点外部流量到达且节点确定 NodePort 或者 xternallPs 服务的后端位于远程节点时，则
该节点通过执行 SNAT 代表其将请求重定向到远程后端。不需要额外的 MTU 更改，而代价是来自后端的答复。


**cilium** 下通过将 **nodeport.mode** 改成 dsr 更改此设置，使 Cilium 的 bpf nodeport实现在 dsr 模式下运行。 

DSR 需要在 **Native-Routing**，不能再任何 **tunneling** 模式下工作。


## XDP
express data path ，能够在网络包进入用户态直接对网络包进行过滤或者处理。XDP 依赖于 ebpf技术。


## Encapsulation（封装）
使用基于 UDP 的封装协议 **vxlan** 形成一个 **tunnel**。

优点简单，缺点是由于添加了 header，MTU过大低于本地路由。导致特定网络连接的最大吞吐率低，可以通过巨型帧缓解。


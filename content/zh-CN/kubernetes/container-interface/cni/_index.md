---
title: "cni"
linkTitle: "CNI"
weight: 2
description: >
  CNI
---

{{% pageinfo %}}
Container Network Interface（CNI）是k8s网络插件的基础。
{{% /pageinfo %}}


Container Network Interface（CNI）是k8s网络插件的基础。基本思想是：Container Runtime 在创建容器时，先创建好 Network Namespace，然后再调用 CNI 插件为这个 net ns 配置网络，
其后再启动容器内的进程。

CNI插件包括两部分：
- 给容器配置网络，有两个基本的接口
    - 配置网络： AddNetwork(net NetworkConfig,rt RuntimeConf)(types.Result,error)
    - 清理网络： DelNetwork(net NetworkConfig,rt RuntimeConf)error
- IPAM Plugin 负责给容器分配 IP 地址，主要实现包括 host-local 和 dhcp


K8s Pod 中的其他容器都是 Pod 所属 pause 容器的网络，创建过程为：
1. kubelet 创建 pause 容器生成 Network Namespace
2. 调用网络 CNI driver
3. CNI driver 根据配置调用具体的 CNI 插件
4. CNI 插件给 pause 容器配置网络
5. pod 中其他的容器都使用 pause 容器的网络


## IPAM

### DHCP
DHCP 插件是最主要的 IPAM 插件之一，用来通过DHCP方式给容器分配 IP 地址。

### host-local
host-local 是常见的 CNI IPAM 插件，用来给 container 分配 IP 地址。

## Flannel
Flannel 通过给每台宿主机分配一个子网的方式为容器提供虚拟网络，使用 UDP 封装IP 包来创建 overlay 网络，借助 etcd 维护网络的分配情况。

## Calico
Calico 是一个基于 BGP 的纯三层的数据中心网络方案，不需要 Overlay。             
Calico 在每个计算节点利用 Linux Kernel 实现了一个高效的 VRouter 负责数据转发，而每个 vRoute 通过 BGP 协议负责把自己上运行的 workload 的路由信息像整个 Calico 网络内传播，大规模下可以通过指定的 BGP route reflector 来完成。保证最终所有的 workload 之间的数据流量都是通过 IP 路由的方式完成互联的。           
Calico 节点组网可以直接利用数据中心的网络结构，不需要额外的 NAT，隧道或者 Overlay Network。

此外，Calico 基于 iptables 提供了丰富的网络Policy。

## Cilium
Cilium 是一个基于 ebpf 和 xdp 的高性能容器网络方案，提供了 CNI 和 CNM 插件。

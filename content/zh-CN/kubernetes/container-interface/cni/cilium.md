---
title: "cilium"
linkTitle: "Cilium"
weight: 1
description: >
  Cilium
---

{{% pageinfo %}}
Cilium
{{% /pageinfo %}}


## cilium路由模式
支持封装路由模式和原生路由模式。

#### 1. 封装路由模式（overlay）
默认的路由模式，在部署时将 **tunnel** 设置为 **vxlan** 或者 **geneve**，它们是两种基于 UDP 的网络封装模式。
在这种模式下，所有集群节点使用基于这样的 UDP 封装协议形成的隧道网络。

#### 2. 原生路由模式（native-routing）
将 **tunnel** 设置为 **disabled**，cilium会将所有不是发往本地另一个端点的数据包交给Linux内核的路由子系统。
如果所有的节点在同一个 L2 网络上，需要添加 **auto-direct-node-routes=true**，不需要额外的组件辅助。
如果节点不再同一个 L2 网络上，需要 BGP daemon 组件的辅助。

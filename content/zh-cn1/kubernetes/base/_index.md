---
title: "k8s名词解释"
linkTitle: "k8s名词解释"
weight: 1
description: >
  kubernetes 名词解释
---

{{% pageinfo %}}
kubernetes 名词解释
{{% /pageinfo %}}


## Kubernetes

先介绍一些名词和定义，方便理解演进的过程与出现的项目。

## CNCF

云原生计算基金会，The Cloud Native Computing Foundation ，简称 **CNCF**。
作为一个厂商中立的基金会，是linxu基金会的一部分。致力于**云原生**应用的推广。如 kubernetes、Argo、Prometheus、Cilium等。

CNCF全景图，源地址：<a href="https://landscape.cncf.io/" target="_blank">landscape</a>

![cloudnativelandscape](/images/k8s/landscape.png)


### CNCF项目成熟度
每个项目都需要有成熟度等级，有三个级别：

- graduated（已毕业）
- incubating（孵化中）
- sandbox（初级项目）

毕业和孵化的项目被认为是稳定的，在生产环境中成功使用的。项目等级是技术委员会采用投票的方式。


## 云原生定义
云原生技术有利于各组织在公有云、私有云和混合云等新型动态环境中，构建和运行可弹性扩展的应用。云原生的代表技术包括容器、服务网格、微服务、不可变基础设施和声明式API。

这些技术能够构建容错性好、易于管理和便于观察的松耦合系统。结合可靠的自动化手段，云原生技术使工程师能够轻松地对系统做出频繁和可预测的重大变更。

## 什么是 kubernetes
一种流行的现代基础设施自动化的开源工具，像一个数据中心的操作系统，管理在 分布式系统 上运行的应用程序。

Kubernetes 在集群的节点上调度容器。它捆绑几个基础设施结构，如应用程序的实例、负载均衡器、持久性存储等，以一种可以被组成应用程序的方式。

- https://github.com/cncf/toc/blob/main/DEFINITION.md#%E4%B8%AD%E6%96%87%E7%89%88%E6%9C%AC
- https://glossary.cncf.io/zh-cn/cloud-native-tech/


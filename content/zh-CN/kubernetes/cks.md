---
title: "cks"
linkTitle: "CKS"
weight: 200
description: >
    cks证书需要在在cka有效期内考取**cks**证书
---

{{% pageinfo %}}
CKS认证考试包括这些一般领域及其在考试中的权重：
- 集群安装：10%	
- 集群强化：15%	
- 系统强化：15%
- 微服务漏洞最小化：20%
- 供应链安全：20%	
- 监控、日志记录和运行时安全：20%
{{% /pageinfo %}}


## 详细内容：
### 集群安装：10%
- 使用网络安全策略来限制集群级别的访问
- 使用CIS基准检查Kubernetes组件(etcd, kubelet, kubedns, kubeapi)的安全配置
- 正确设置带有安全控制的Ingress对象
- 保护节点元数据和端点
- 最小化GUI元素的使用和访问
- 在部署之前验证平台二进制文件


### 集群强化：15%
- 限制访问Kubernetes API
- 使用基于角色的访问控制来最小化暴露
- 谨慎使用服务帐户，例如禁用默认设置，减少新创建帐户的权限
- 经常更新Kubernetes



### 系统强化：15%
- 最小化主机操作系统的大小(减少攻击面)
- 最小化IAM角色
- 最小化对网络的外部访问
- 适当使用内核强化工具，如AppArmor, seccomp



### 微服务漏洞最小化：20%
- 设置适当的OS级安全域，例如使用PSP, OPA，安全上下文
- 管理Kubernetes机密
- 在多租户环境中使用容器运行时 (例如gvisor, kata容器)
- 使用mTLS实现Pod对Pod加密



### 供应链安全：20%
- 最小化基本镜像大小
- 保护您的供应链：将允许的注册表列入白名单，对镜像进行签名和验证
- 使用用户工作负载的静态分析(例如kubernetes资源，Docker文件)
- 扫描镜像，找出已知的漏洞



### 监控、日志记录和运行时安全：20%
- 在主机和容器级别执行系统调用进程和文件活动的行为分析，以检测恶意活动
- 检测物理基础架构，应用程序，网络，数据，用户和工作负载中的威胁
- 检测攻击的所有阶段，无论它发生在哪里，如何扩散
- 对环境中的不良行为者进行深入的分析调查和识别
- 确保容器在运行时不变
- 使用审计日志来监视访问

## 真题模拟

### 1. Pod 指定 ServiceAccount
Task 1.在现有namespace qa中创建一个名为 backend-sa 的新 ServiceAccount ，确保此 ServiceAccount 不自动挂载 API 凭据。
2. 使用 /cks/sa/pod1.yaml 中的清单文件来创建一个 Pod
3. 最后，清理 namespace qa 中任何未使用的 ServiceAccount

```
k create sa -n qa backend-sa
k explain serviceaccount  #获取不挂载API的字段
k edit sa backend-sa -n qa  # 复制进来，值为 false

k apply -f /cks/sa/pod1.yaml

k get pod -oyaml -n qa|grep serviceAccont:  #查看所有使用的sa
k dekete sa -n qa xxx #清理

```

### 2.  RBAC - Rolebinding
Context 绑定 Pod 的 ServiceAccount 的 Role授予宽松的权限，完成以下项目减少权限集合：
Task:  一个名为 web-pod 的现有 Pod 在 namespace db 中运行，编辑绑定到pod 的 serviceaccount service-account-web 的现有的 Role,
仅允许对 services 类型的资源执行 get 操作，在 namespace db 中创建一个名为 role-2 ，并仅允许对namespace类型的资源执行 delete 操作的新 role。
创建一个名为 role-2-binding 的新 rolebinbing，将创建的 Role 绑定到 Pod 的serviceaccount。
注意： 请勿删除现有 rolebinding 。


```
k get role -n db
k edit role -n db xxx # 对services执行get操作

k create role role-2 -n db --verb=delete --resource=namespaces
k create rolebinding  role-2-binding -n db --role=role-2 --serviceaccount=db:service-account-web

#验证
k auth can-i get services -n db --as=system:serviceaccount:db:service-account-web
k auth can-i delete namespaces -n db --as=system:serviceaccount:aba:service-account-web

https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/authorization/

```


### 3. 启用 API server 认证
由 kubeadm 创建的cluster 的k8s api 服务器，出于测试目的，临时配置允许未经身份验证和未经授权的访问，授予匿名用户 cluster-admin 权限。

Task: 重新配置 cluster 的 kubernetes api 服务器，以确保只允许经过身份验证和授权的rest请求。
使用授权模式Node，RBAC和准入控制器 NodeRestriction。
删除用户 system:anonymous 的 Clusterrolebinding 来进行清理。

注意： 所有kubectl配置的未授权的访问，必须更改。一旦完成 cluster 的安全加固，kubectl的配置将无法工作，可以使用master节点上的配置文件
 /etc/kubernetes/admin.conf，以确保经过身份验证的授权的请求仍然内允许。

```
# 修改配置文件
--authorization-mode RBAC,Node
--enable-admission-plugins NodeRestriction

k delete clusterRolebinding system:anonymous

https://kubernetes.io/zh-cn/docs/reference/command-line-tools-reference/kube-apiserver/

#切换到master机器上
kubectl get node -owide --kubeconfig=/etc/kubernetes/admin.conf

```

### 4. Sysdig & falco
Task : 使用运行时检测工具来检测Pod tomcat 单个容器中频发生成和执行的异常进程。有两种工具可供使用：
- sysdig
- falco

注意： 这些工具只预装在 cluster 的工作节点，不再master 节点。使用工具至少分析 30 秒， 使用过滤器检查生成和执行的进程，将事件写到
 /opt/JSR00101/incidents/summary 文件中，其中包含检测的事件。

格式如下： [timestamp],[uid],[processName] 保持工具的原始时间戳格式不变。

注意：确保事件文件存储在集群的工作节点上。



### 5.  容器安全，删除特权pod

---
title: "cks"
linkTitle: "CKS"
weight: 500
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

# 好像要用上面的sa，先edit pod1.yaml 加上 serviceAccountName: backend-sa
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

#验证，可以使用 describe 或者 auth can-i
k describe rolebinding role-2-binding -n db

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

注意： 这些工具只预装在 cluster 的工作节点，不在master 节点。使用工具至少分析 30 秒， 使用过滤器检查生成和执行的进程，将事件写到
 /opt/JSR00101/incidents/summary 文件中，其中包含检测的事件。

格式如下： [timestamp],[uid],[processName] 保持工具的原始时间戳格式不变。

注意：确保事件文件存储在集群的工作节点上。






### 5.  默认网络策略
Context           
一个默认拒绝（default-deny）的 NetworkPolicy 可避免在未定义任何其他 NetworkPolicy 的 Namespace 中意外公开pod。

Task:               
为所有类型为 Ingress + Egress 的流量在 namespace **testing** 中创建一个名为 denypolicy 的新默认拒绝 NetworkPolicy。
此新的 NetworkPolicy 必须拒绝 namespace testing 中的所有的 Ingress + Egress 流量。

将新创建的默认拒绝 NetworkPolicy 应用与在 namespace testing 中运行的所有的pod。

```yaml
# 拒绝所有pod 
# https://kubernetes.io/zh-cn/docs/concepts/services-networking/network-policies/#default-deny-all-ingress-and-all-egress-traffic
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: denypolicy
  namespace: testing
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress

```

```shell
# 应用
kubectl apply -f net.yaml
# 检查
kubectl describe networkpolicy -n testing denypolicy
```

### 6. 审计日志 log audit
在 cluster 中启用审计日志。为此，请启用日志后端，并确保：
- 日志存储在/var/log/kubernetes/audit-logs.txt
- 日志文件能保存 10 天
- 最多保留 2 个旧审计日志文件


/etc/kubernetes/logpolicy/sample-policy.yaml 提供了基本策略。它仅指定**不记录**的内容。
注意：基本策略位于 cluster 的 master 节点上。

编辑和扩展基本策略以记录：
- RequestResponse 级别的 **persistentvolumes** 更改
- namespace **front-apps** 中 **configmaps** 更改的请求体
- Metadata 级别的所有 namespace 中的 **Configmap** 和 **secret** 的更改

此外，添加一个全方位的规则以在 Metadata 级别记录所有其他请求。


注意： 不要忘记应用修改后的策略。

https://kubernetes.io/zh-cn/docs/tasks/debug/debug-cluster/audit/#audit-policy


```yml
apiVersion: audit.k8s.io/v1
kind: Policy
omitStages:
  - ReqyestReceived
rules:
  - level: RequestResponse
    resources:
    - group: ""
      resources: ["persistentvolumes"]
  - level: Request
    resources:
    - group: ""
      resources: ["configmaps"]
    namespaces: ["front-apps"]
  - level: Metadata
    resources:
    - group: ""
      resources: ["recrets","configmaps"]
  - level: Metadata
    omitStages:
      - "RequestReceived"
```

备份 kube-apiserver.yaml 文件，然后修改，新增一下内容：

```
- --audit-log-path: /var/log/kubernetes/audit-logs.txt
- --audit-log-maxage: 10
- --audit-log-maxbackup: 2
- --audit-policy-file=/etc/kubernetes/logpolicy/sample-policy.yaml


volumeMounts:
    - mountPath: /var/log/kubernetes
    name: audit-log
    readOnly: false
    - mountPath: /etc/kubernetes/logpolicy
    name: audit
    readOnly: true
volumes:
    - hostPath:
        path: /var/log/kubernetes
        type: DirectoryOrCreate
    name: audit-log
    - hostPath:
        path: /etc/kubernetes/logpolicy
        type: DirectoryOrCreate
    name: audit
```

重启 kubelet
```shell
# 重启
systemctl daemon-reload
systemctl restart kubelet

#检查
tail -f /var/log/kubernetes/audit-log.txt
```


## 7. 创建Secret
在 namespace istio-system 中获取名为 db1-test 的现有 secret 的内容。
将 username 字段存储在  /cka/sec/user.txt 文件中，并将 Password 字段存储在名为 /cks/sec/pass.txt 文件中。

注意： 不要再一下步骤中使用/修改先前创建的文件，如果需要，可以创建新的临时文件。


在 istio-system namespace 中创建一个名为 db2-test 的新 secret ，内容如下：
username: adf
password: KvLft

最后，创建一个新的pod，可以通过卷访问secret db2-test：

- 容器名dev-container 
- 镜像 nginx 
- 卷名 secret-volume 
- 挂载路径 /etc/secret


https://kubernetes.io/zh-cn/docs/concepts/configuration/secret/#restriction-secret-must-exist


```shell
k get secret db1-test -n istio-system
mkdir -p /cks/sec
cd /cks/spc
echo 值 |base64 -d > user.txt
echo 值 |base64 -d > pass.txt
cat user.txt
cat pass.txt
```

```
k create secret generic db2-test -n istio-system --from-literal=username=adf --from-literal=password=KvLft

搜 secret ,复制模板改要求即可。
```

## 8. dockerfile 检测
分析和编辑给定的 Dockerfile **/cks/docker/Dockerfile**，并修复在文件中拥有的突出的安全/最佳实践问题的两个指令。

分析和编辑给定清单文件 /cks/docker/deployment.yaml ，并修复在文件中突出的安全/最佳实践问题的两个字段。

注意：请勿删除或者添加配置设置；只需修改现有的配置设置，让两个配置不再有安全问题。

注意： 如果需要非特权用户来执行任何项目，请使用用户ID **65535** 的用户 **nobody**。


只需要修改即可，不需要创建。

```
Dockerfile 看题目，image的tag不正确，然后需要将 USER root 换成 **USER nobody**

Deployment 文件将 SecurityContext 里面的admin权限删掉：**SYS_ADMIN**，第二个是 pod 的 label标签和 matchLabels 不匹配，修改 pod 的 metadata labels。
```

## 9. 沙箱运行容器 gVisor
该 cluster 使用 containerd 作为 CRI 运行时，containerd 的默认运行时处理程序是runc。
Containerd 已准备好支持额外的运行时处理程序 runsc（gVisor）

Task：                           
使用名为 runsc 的现有运行时处理程序，创建一个名为 **untrusted** 的 RuntimeClass。
更新 namespace server 中的所有pod以在 gVisor 上运行。

可以在 /cks/gVisor/rc.yaml 中找到一个模板清单。

```
# 创建 runtimeclass，官网搜索模板

apiVersion: node.k8s.io/v1
kind: RuntimeClass
metadata:
  # 用来引用 RuntimeClass 的名字
  # RuntimeClass 是一个集群层面的资源
  name: untrusted
# 对应的 CRI 配置的名称
handler: runsc


# 查看pod，是否是修改deploy，添加字段并验证
runtimeClassName: untrusted

```

## 10. 容器安全，删除特权pod
最佳实践是将容器设计为无状态和不可变的。

Task：检查在 namespace production 中运行的pod，并删除任何非无状态或者非不可变的pod。

使用以下对无状态和不可变的严格解释：
- 能够在容器内存储数据的 Pod 容器必须被视为非无状态的。
- 被配置为任何形式的特权 Pod 必须被视为可能是非无状态和非不可变的。

**思路：**               
要求删除非无状态和非不可变的pod，即删除任何形式的特权pod和挂载volume的pod。

```bash
# 查询是不是pod控制
k get all -n production
# 查询pod
k get pod -n production

# 每个pod查看yaml，如果满足要求直接删除
k get pod -n production -oyaml xxx
k delete pod -n production xxx

```


## 11. container 安全上下文
**Context**

Container Security Context 应在特定 namespace 中修改 Deployment


**Task**

按照如下要求修改 **sec-ns** 命名空间中的 Deplyment **secdep**

1. 用 ID 为 30000 的用户启动容器（设置用户ID为 30000）
2. 不允许进程获取超出其父进程的特权（禁止 allowPrivilegeEscalation）
3. 以只读方式加载容器的根文件系统（对根文件的只读特权）

> k create deploy secdep --image=nginx --replicas=2 --dry-run=client -oyaml  -n sec-ns


```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: secdep
  name: secdep
  namespace: sec-ns
spec:
  securityContext:  # 添加
    runAsUser: 30000
  replicas: 2
  selector:
    matchLabels:
      app: secdep
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: secdep
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
        securityContext:  # 添加，如果有多个容器，每个容器都要添加
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true

```

## 12. 网络策略 NetworkPolicy
Task              
创建一个名为 pod-restriction 的 NetworkPolicy 来限制对在 namespace **dev-team** 中运行的 Pod products-service 的访问。 

只允许以下 Pod 连接到 Pod products-service 
- namespace **qa** 中的 Pod 
- 位于任何 namespace，带有标签 **environment: testing** 的 Pod

注意：确保应用 NetworkPolicy。 你可以在/cks/net/po.yaml 找到一个模板清单文件。

**思路：**
1. 查看pod的label
2. 使用ingress，查看 ns **qa** 的label
3. 最后apply 

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: pod-restriction
  namespace: dev-team
spec:
  podSelector:
    matchLabels:
      name: products-service
  policyTypes:
    - Ingress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              name: qa
    - from:
        - podSelector:
            matchLabels:
              environment: testing
          namespaceSelector: {}
```

```shell
# apply 和验证
k create -f netpol.yaml
k describe networkpolicy pod-restriction -n dev-team
```

## 13. Trivy 扫描镜像安全漏洞
**Task** 
使用 Trivy 开源容器扫描器检测 namespace **kamino** 中 Pod 使用的具有严重漏洞的镜像。 

查找具有 High 或者 Critical 严重性漏洞的镜像，并删除使用这些镜像的 Pod。

注意：Trivy仅安装在cluster的master节点上，在工作节点上不可使用。
必须切换到cluster的master节点才能使用Trivy.


```
trivy image --help

trivy image --severity HIGH,CRITICAL  image:tag

k get pods --namespace kamino --output=custom-columns="Name:.metadata.name,Img:.spec.containers[*].image"
k delete pod -n kamino xxx
```

## 14. AppArmor 
Context                
AppArmor 已在 cluster 的工作节点上被启用，一个 AppArmor 配置文件已存在，但尚未被实施。

Task                 
在 cluster 的工作节点上，实施位于 **/etc/apparmor.d/nginx_apparmor** 的现有 AppArmor 配置文件。
编辑位于 **/home/candidate/KSSK00401/nginx-deploy.yaml** 的现有清单文件以应用 AppArmor 配置文件。最后，应用清单文件并创建其中指定的Pod。



## 15. 





https://www.cnblogs.com/huss2016/p/17055905.html
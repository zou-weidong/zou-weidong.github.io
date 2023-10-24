---
title: "访问apiserver"
linkTitle: "访问apiserver"
weight: 219
description: >
    如何访问apiserver？
---

# 1. 在集群外访问apiserver

## 1. kubectl 访问 apiserver
默认配置在 **$HOME/.kube/config** ,或者指定配置文件 **kubectl --kubeconfig** 。

可以使用环境变量 **KUBECONFIG** 。

```
--v=0   Generally useful for this to ALWAYS be visible to an operator.
--v=1   A reasonable default log level if you don’t want verbosity.
--v=2   Useful steady state information about the service and important log messages that may correlate to significant changes in the system. This is the recommended default log level for most systems.
--v=3   Extended information about changes.
--v=4   Debug level verbosity.
--v=6   Display requested resources.
--v=7   Display HTTP request headers.
--v=8   Display HTTP request contents
```

## 2.curl 访问 apiserver

```shell
# 1. 获取token
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: custome-sa
  namespace: default
---
apiVersion: v1
kind: Secret
metadata:
  name: custome-sa
  namespace: default
  annotations:
    kubernetes.io/service-account.name: "custome-sa"
  namespace: default
type: kubernetes.io/service-account-token
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: sa-crb
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: custome-sa
    namespace: default
EOF

TOKEN=`kubectl get secret custome-sa --template={{.data.token}} | base64 --decode`
echo $TOKEN
```

```shell
#2. 直接访问
curl --insecure -H "Authorization: Bearer $TOKEN" https://xxx:443/api/v1/namespaces/default/services

# 或者可以使用本地代理访问
kubectl proxy --port 8080
curl --insecure -H "Authorization: Bearer $TOKEN" https://127.0.0.1:8080/api/v1/namespaces/default/services
```

# 2. 集群内访问 apiserver
在 Pod 内通过 serviceaccount 内访问。

```shell
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: custome-sa
  namespace: default
---
apiVersion: v1
kind: Secret
metadata:
  name: custome-sa
  annotations:
    kubernetes.io/service-account.name: "custome-sa"
  namespace: default
type: kubernetes.io/service-account-token
---

kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: pods-list
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list"]

---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: pods-list
subjects:
- kind: ServiceAccount
  name: custome-sa
  namespace: default
roleRef:
  kind: ClusterRole
  name: pods-list
  apiGroup: rbac.authorization.k8s.io
EOF

TOKEN=`kubectl get secret custome-sa --template={{.data.token}} | base64 --decode`
echo $TOKEN

```

```yaml
# 创建工作负载时指定sa
spec：
  serviceAccountName: custome-sa
```

```shell
# 在 pod 内访问
export CURL_CA_BUNDLE=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
curl -H "Authorization: Bearer $TOKEN" https://kubernetes
```

# 3. Client-go 访问 api
- RestClient 客户端      对httpclient的封装
- ClientSet 客户端       仅支持内置资源(最常用)
- DynamicClient 客户端   支持内置资源 + crd
- DiscoveryClient 客户端

![tls](/images/k8s/access-apiserver.png)


Client-go 目录结构：
- discovery: 提供 DisconveryClient 发现客户端。
- dynamic: 提供 DynamicClient 发现客户端。
- kubernetes: 提供 ClientSet 客户端。
- rest: 提供 RESTClient 发现客户端。
- informers: 每种 Kubernetes 资源的 Informer 实现。
- listers: 为每一个 Kubernetes 资源提供 Lister 功能，提供只读缓存数据。
- plugin: 提供云服务商授权插件。
- scale: 提供 ScaleClient 发现客户端。
- tools: 提供常用工具。
- transport: 提供安全的 TCP 连接。
- util: 提供常用方法。


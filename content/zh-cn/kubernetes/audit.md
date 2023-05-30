---
title: "审计"
linkTitle: "审计"
weight: 181
description: >
    (k8s集群审计)[https://kubernetes.io/zh-cn/docs/tasks/debug/debug-cluster/audit/#audit-policy]
---

{{% pageinfo %}}
Kubernetes 审计（Audit）提供了安全相关的时序操作记录，支持 **日志** 和 **webhook** 两种格式，并可以通过审计策略自定义事件类型。
{{% /pageinfo %}}

kubernetes 审计功能提供了与安全相关的、按照事件顺序排列的记录集，记录每个用户、使用 kubernetes API 的应用以及控制面自身引发的活动。

审计记录最初产生于 kube-apiserver 内部，每个请求在不同阶段都会生成审计事件，这些事件会根据特定策略被预处理并写入后端。
策略确定要记录的内容和用来存储记录的后端。


每个请求都可被记录其相关的阶段（stage），已定义的阶段有：
- RequestReceived 审计处理器接收到请求后，并且在委托给其余处理器之前发生的事件。
- ResponseStarted 在响应消息的头部发送后，响应消息体发送前发生的事件。只有长时间运行的请求（例如 watch） 才会有这个阶段
- ResponseComplete 在响应消息体完成并且没有更多数据需要传输的时候。
- Panic  当 panic 发生时生成 

## 审计策略：
审计策略定义了相关应记录哪些事件以及哪些数据的规则。审计策略对象结构定义在 **audit.k8s.io** API组。处理事件时，按照顺序与规则列表进行比较。

第一个匹配规则设置事件的 审计级别（Audit Level）。已定义的审计级别有：
- None 符合这条规则的日志将不会记录
- Metadata 记录请求的元数据，但没有消息体
- Request 记录事件的元数据和请求的消息体，但是不记录响应的消息体
- RequestResponse 记录事件的元数据，请求和响应的消息体，不适用于非资源类型请求


> 使用 **--audit-policy-file**  参数指定策略，传递给 kube-apiserver 。如果不设置该标志，则不记录事件。
> 注意 rules 字段必须在审计策略文件中提供，没有规则的策略将被视为非法配置。




### 审计日志 Log 后端
配置 **kube-apiserver** 的参数开启审计日志。
- audit-log-format：日志格式
- audit-log-path：审计日志路径
- audit-log-maxage：旧日志最长保留天数
- audit-log-maxbackup：旧日志文件最长保留个数
- audit-log-maxsize：日志文件最大大小（单位MB），超过后自动做轮转（默认为100MB）

eg：

    --audit-log-format=json
    --audit-log-maxage=30
    --audit-log-maxbackup=10
    --audit-log-maxsize=100
    --audit-log-path=/var/log/audit/kube-audit.log
    --audit-policy-file=/etc/kubernetes/policies/audit-policy.yaml

eg:

```
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"5737a616-9f5a-4efe-98f8-90f5b1b56880","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/default/configmaps/console-operator-lock","verb":"get","user":{"username":"system:serviceaccount:default:default-sa","uid":"1640a9e4-eef5-4b39-8d6d-871fef41107b","groups":["system:serviceaccounts","system:serviceaccounts:default","system:authenticated"],"extra":{"authentication.kubernetes.io/pod-name":["console-operator-7bd67974fd-bvxf4"],"authentication.kubernetes.io/pod-uid":["275b6b2a-f58f-4443-8544-e277c24138a0"]}},"sourceIPs":["10.222.8.92"],"userAgent":"console-operator/v0.0.0 (linux/amd64) kubernetes/$Format/leader-election","objectRef":{"resource":"configmaps","namespace":"default","name":"console-operator-lock","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2023-04-06T01:57:25.529046Z","stageTimestamp":"2023-04-06T01:57:25.538347Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":"RBAC: allowed by ClusterRoleBinding \"default-sa\" of ClusterRole \"default-sa\" to ServiceAccount \"default-sa/default\""}}
```








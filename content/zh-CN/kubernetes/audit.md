---
title: "审计"
linkTitle: "审计"
weight: 181
description: >
    k8s集群审计
---

{{% pageinfo %}}
Kubernetes 审计（Audit）提供了安全相关的时序操作记录，支持 **日志** 和 **webhook** 两种格式，并可以通过审计策略自定义事件类型。
{{% /pageinfo %}}


## 审计日志
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

## 审计策略：
使用 **--audit-policy-file**  参数指定策略
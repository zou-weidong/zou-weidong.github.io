---
title: "k8s集群备份"
linkTitle: "集群备份"
weight: 180
description: >
    k8s集群备份
---

{{% pageinfo %}}
使用 **velero** 进行k8s集群和持久卷的备份、迁移和灾难恢复。
{{% /pageinfo %}}


## 安装
可以使用 **velero cli** 或者 **helm** 进行安装。

- https://github.com/heptio/velero/releases
- https://velero.io/
- https://vmware-tanzu.github.io/helm-charts/


启动 velero ，创建定期备份，一般每天凌晨备份一次，可以忽略持久化备份。
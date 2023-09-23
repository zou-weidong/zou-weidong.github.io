---
title: "CSI"
linkTitle: "CSI"
weight: 1
description: >
  CSI
---

{{% pageinfo %}}
Container Storage Interface (CSI)。在 kubernetes V1.13 版本中升级为 GA 。
{{% /pageinfo %}}


CSI 为容器编排系统（如 kubernetes）定义标准接口，以将任意存储系统暴露给它们的容器工作负载。


CSI 卷类型是一种 out-tree 的 CSI 卷插件，用于 Pod 与同一节点上运行的外部 CSI 卷驱动程序交互。
部署 CSI 兼容卷驱动后，用户可以使用 csi 作为卷类型来挂载驱动提供的存储。

CSI 卷可以在 Pod中以三种方式使用：
1. 通过 PVC 对象引用
2. 使用一般性的临时卷
3. 使用 CSI临时卷，前提时驱动支持这种用法


CSI 持久卷具有以下字段可供用户指定：
- driver：一个字符串，指定要使用的卷驱动程序的名称。
- volumeHandle：一个字符串，唯一标识从 CSI 卷插件的 CreateVolume 调用返回的卷名。随后在卷驱动程序的所有后续调用中使用卷句柄来引用该卷。
- readOnly：可选的布尔值，指定卷是否被发布为只读，默认是false。

##  卷插件类型
[卷插件常用问题](https://github.com/kubernetes/community/blob/master/sig-storage/volume-plugin-faq.md)

### 树外（Out-of-Tree）卷插件
Out-of-Tree 卷插件包括容器存储接口CSI。它们使存储供应商创建自定义存储插件，而无需将插件源码添加到 kubernetes 代码仓库。

CSI 允许独立于 Kubernetes 代码库开发卷插件，并作为扩展部署（安装）在 Kubernetes 集群上。

### 树内（In-Tree）
以前所有的卷插件都是树内的。“树内”插件是与 Kubernetes 的核心组件一同构建、链接、编译和交付的。这意味着向 Kubernetes 添加新的存储系统（卷插件）需要将
代码合并到 kubernetes 核心代码库中。


## 动态配置
通过为 CSI 创建插件 **StorageClass** 来支持动态配置的 CSI Storage 插件启用自动创建/删除。


例如，以下允许通过名为 **com.example.team/csi-driver** 的 CSI Volume Plugin 动态创建 fast-storage 的 Volume。

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: fast-storage
provisioner: com.example.team/csi-driver
parameters:
  type: pd-ssd
```

要触发动态配置，需要创建一个 **PersistentVolumeClaim** 对象。例如，下面的 pvc 可以使用上面的 storageClass 触发动态配置。

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-request-for-storage
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: fast-storage
```

当动态创建 Volume 时，通过 CreateVolume 调用，将参数 type: pd-ssd 传递给 CSI 插件 **com.example.team/csi-driver**。
作为响应，外部 volume 插件会创建一个新的 Volume，然后自动创建一个 pv 对象来对应前面的 PVC。
然后，kubernetes 会将新的 PersistentVolume 对象绑定到 pvc，使其可以使用。


## 创建 CSI 驱动


---
title: "准入控制器"
linkTitle: "准入控制器"
weight: 500
description: >
    准入控制器
---

{{% pageinfo %}}
准入控制器 是一段代码，它会在请求通过认证和鉴权之后、对象被持久化之前拦截到达 API 服务器的请求。
{{% /pageinfo %}}


准入控制器可以执行**验证（Validating）**和**变更（Mutating）**操作。变更控制器可以根据被其接受的请求更改相关对象；验证控制器则不行。

准入控制器限制创建、删除、修改对象的请求。不会阻止读取（get watch list）对象的请求。


## 准入控制阶段
两个阶段。第一阶段，运行变更准入控制器。第二阶段，运行验证准入控制器。

如果两个阶段之一的任何一个控制器拒绝了某请求，则整个请求将立即被拒绝，并向最终用户返回错误。


> 启用：  --enable-admission-plugins=NamespaceLifecycle,LimitRanger ...

> 关闭： --disable-admission-plugins=PodNodeSelector,AlwaysDeny ...



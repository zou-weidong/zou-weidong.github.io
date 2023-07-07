
---
title: "gatekeeper"
linkTitle: "gatekeeper"
description: >
    gatekeeper
---


# GateKeeper
OPA gatekeeper 是一个专业项目，提供 OPA 和 kubernetes 之间的一些集成。

OPA GateKeeper 在普通 OPA 之上添加了以下内容：
1. 一个可扩展的参数化策略库
2. 用于实例化约束的原生 CRD
3. 用于扩展的策略库的云原生kubernetes CRD


Kubernetes API 服务器被分配为在创建、更新、删除对象时查询 OPA 以准入控制决策。

ApiServer 将 webhook 请求中的整个kubernetes对象发送给 OPA ，OPA 使用准入审查来评估加载的策略 input。


Input 文档包含以下字段：

- input.request.kind       类型，eg Pod，Service
- input.request.operation  操作,eg  CREATE UPDATE DELETE CONNECT
- input.request.userInfo   身份信息
- input.request.object     包含完整的 kubernetes 对象
- input.request.oldObject  在操作是 UPDATE 和  DELETE 之前的object





---
title: "opa"
linkTitle: "OPA"
weight: 50
description: >
  OPA 提供了一种高级声明性语言，可以将策略指定为代理和简单的API。可以将 OPA 实施在 微服务、kubernetes、CI/CD管道、API网关等中实施策略。
---

{{% pageinfo %}}
OPA 是一种轻量级通用策略引擎，可以与服务共处一地。可以将OPA作为 sidecar、主机级守护程序或者库进行集成。
{{% /pageinfo %}}



## 概述
OPA 将策略指定和策略执行分开。当需要做出决策时，会查询 OPA 提供的结构化数据作为输入，OPA 接受任意结构化数据作为输入。

策略通常有：
- 允许哪些用户可以访问哪些资源
- 允许哪些子网出口流量
- 必须将工作负载部署在哪些集群
- 可以从哪些注册表二进制文件下载
- 容器可以执行哪些操作系统功能

表达式：
- 逻辑与
- and   用 ；连接在一起，也可以通过多行来省略（AND）运算符
- 不为真 


变量：
- 使用赋值运算符将值存储在中间变量中 **:=**
```
s := input.servers[0]
s.id == "app"
p := s.protocols[0]
p == "https"
```

当 OPA 计算表达式时，会找到使所有表达式都为真的变量的值。如果没有给所有为真的变量赋值，则结果未定义。

变量是不可变的，如果尝试两次分配同一个变量。OPA会报告错误。

opa必须要能够枚举表达式总和功能所有变量的值。如果OPA无法枚举任何表达式中变量的值，则opa报错。

迭代：
- 当将变量注入表达式时，Rego中的迭代会隐式发生。
```
some i : input.networks[i].public == true
# 返回的 i 即为数组的下标


some i,j : input.services[i].protocols[j] == "http"
# 返回 i 和 j 的下标

some i,j
id := input.ports[i].id

```

```
public_network contians net.d if{
    some net in  input.networks
    net.public
}

shell_accessible containers server.id if{
    some server in input.servers
    "telnet" in server.protocols
}

shell_accessible contians server.id if{
    some server in input.servers
    "ssh" in server.protocols
}

```


对于所有 every
```
no_telnet_exposed if {
    every server in input.servers  {
        every protocol in server.portocols {
            "telnet" != protocol
        }
    }
}

```

## Rules
Rego 允许使用规则封装和重用逻辑。规则只是如果/那么逻辑语句，规则可以是完整的或者部分的。

### 完整规则
```
any_public_networks := true if {    # head
    sone net in input.networks      # body
    net.public
}

```

### 部分规则
```
public_network contains net.id if{
    some net in input.networks
    net.public
}
```

## 逻辑 or
input 的 servers 中只要有一个就是正确的
```
package example.logical_or

default shell_accessible := false

shell_accessible := true {
    input.servers[_].protocols[_] == "telnet"

}

shell_accessible := true {
    input.servers[_].protocols[_] == "ssh"
}

```


## Rego
使用 Rego 定义易于读写的策略。
Rego 专注于为引用嵌套文档提供强大的支持，并确保查询正确无误。
Rego 是声明性的，因此策略作者可以专注于应该返回什么查询，而不是应该如何执行查询。这些查询比命令式语言中的等效查询更简单、更简洁。
与支持声明式查询语言的其他应用程序一样，OPA能够优化查询以提高性能。


基本语法：
- 上下文 data
- 输入 input
- 索引取值 data.bindings[0]
- 比较 alice == input.subject.user
- 赋值 user := input.subject.user
- 规则  	< Header > { < Body > }
- 规则头	< Name > = < Value > { … } 或者 < Name > { … }
- 规则体	And运算的一个个描述
- 多条同名规则	Or运算的一个规则
- 规则默认值	default allow = false
- 函数	fun(x) { … }
- 虚拟文档	doc[x] { … }

输入会挂在 input 对象下，用到的上下文会挂在data对象下。

rule：
当定义规则时：
- 每条规则都会有返回值，不声明返回值则只返回 true 或者 false，声明返回值则返回其值。
- 规则体内每条描述都会逐条 And 运算，全部成立才会返回值
- 多条同名规则相互之间是 or 运算，满足其一即可

具体到代码中规则 allow ，默认值是 false
要求 user_has_role 和 role_has_permission 同时满足，两者的 role_name 也是一样。

你可能发现，局部变量 role_name 没声明，Rego里可以省略声明局部变量，直接使用。

其中 user_has_role[role_name] 这种带参数的结构不是规则，叫**虚拟文档**。






---
title: "jdbc"
linkTitle: "jdbc"
weight: 4
description: >
  什么是数据库连接池，解决什么问题？
---

{{% pageinfo %}}
连接池基本的思想是在系统初始化的时候，将数据库连接作为对象存储在内存中。当用户需要访问数据库时，从池中取出一个已建立的空闲连接对象。使用完后放回池中，以供下一个请求访问使用。
{{% /pageinfo %}}


# springboot 默认连接池 hikari 配置

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/test?allowPublicKeyRetrieval=true&characterEncoding=utf8&useSSL=false&serverTimezone=Asia/Shanghai
    username: root
    password: xxx
    type: com.zaxxer.hikari.HikariDataSource
    hikari:
      # 连接池名称
      pool-name: pool-demo
      # 最小连接数，一直保持的连接
      minimum-idle: 5
      # 空闲连接存活最大时间，默认10分钟
      idle-timeout: 600000
      # 最大连接数，默认20
      maximum-pool-size: 20
      # 连接池中连接的最长生命周期，默认30分钟
      max-lifetime: 1800000
      # 数据库连接超时时间，默认30秒
      connection-timeout: 30000
      # 此属性控制从池返回的连接的默认自动提交行为,默认值：true
      auto-commit: true
      connection-test-query: select 1
```

## 为什么称为默认连接池：
- 字节码精简
- 优化代理和拦截器
- 自定义数组类型（FastStatementList 替代 ArrayList）
- 自定义集合类型（ConcurrentBag）
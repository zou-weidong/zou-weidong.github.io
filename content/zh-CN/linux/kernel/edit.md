---
title: "修改内核参数"
linkTitle: "修改内核参数"
weight: 2
description: >
  查看、修改 Linux 内核参数
---

{{% pageinfo %}}
查看、修改 Linux 内核参数
{{% /pageinfo %}}


## 查看
### 方法一：
通过 **/proc/sys** 目录，使用 cat 命令查看对应文件的内容。           
此目录是 Linux 内核启动后生成的伪目录，其目录下的 net 文件夹中存放了当前系统中生效的所有内核参数、目录树结构与参数的完整名称相关。
如 **net.ipv4.tcp_tw_recycle** ，对应的文件就是 **/proc/sys/net/ipv4/tcp_tw_recycle**，文件的内容就是参数值。
执行如下命令查看：

> cat /proc/sys/net/ipv4/tcp_tw_recycle

### 方法二：
通过 **/etc/sysctl.conf** 文件进行查看，执行以下命令，查看当前系统中生效的所有参数。

> sysctl -a

## 修改
- 方法一：通过修改 /proc/sys 目录中对应的文件，临时修改，重启后会重置为原参数值，一般用于临时性验证。
- 方法二：通过修改 /etc/sysctl.conf 文件进行修改，该方法修改的参数值，永久生效。
    1. 修改执行的参数值： sysctl -w kernel.domainname="example.com"
    2. 修改配置文件： vim /etc/sysctl.conf
    3. 配置生效： sysctl -p

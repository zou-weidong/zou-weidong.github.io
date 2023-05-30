---
title: "操作网络命令"
linkTitle: "操作网络命令"
weight: 3
description: >
  操作网络命令
---

{{% pageinfo %}}
Linux 下控制网络的命令：
- curl & wget 
- ping
- tracepath & traceroute
- mtr
- host
- whois
- ifplugstatus 是否有网线插入网络设备接口， sudo apt-get install ifplugd
- ifconfig
- ifdown & ifup  启用或者禁用设备
- dhclient
- netstat
{{% /pageinfo %}}


## route 命令
route 命令用于显示和操作 IP路由表。          
要实现两个不同的子网之间的通信，需要一台连接两个网络的路由器，或者同时位于两个网络的网关来实现。

在Linux系统中，设置路由通常是为了解决以下问题：
Linux机器在局域网中，局域网有一个网关，能够让机器访问 Internet ，那么需要将这台机器的IP地址设置为Linux机器的默认路由。
直接执行route命令来添加路由，不会永久保存。当网卡重启或者机器重启之后，该路由会失效；可以在 /etc/rc.local 中添加 route 命令来保证该路由设置永久有效。


命令参数：

- -c 显示更多信息
- -n 不解析名字
- -v 显示详细的处理信息
- -F 显示发送信息
- -C 显示路由缓存
- -f 清除所有网关入口的路由表
- -p 与 add 命令一起使用时使路由具有永久性 



- add：添加一条路由
- del：删除一条路由
- -net：目标地址是一个网络
- -host：目标地址是一个主机
- netmask：当添加一个网络路由时，需要使用网络掩码
- gw：路由数据包通过网关。注意，指定的网关必须能够达到。
- metric：设置路由跳数
- Command 指定想运行的命令 （Add/Change/Delete/Print）
- Destination 该路由的网络目标
- mask Netmask 指定与网络目标的网络掩码（子网掩码）
- Gateway 指定网络目标定义的地址集和子网掩码可以到达的前进或者下一跳跃点 IP 地址
- metric Metric为路由指定一个整数成本值，当在路由表的多个路由中进行选择时可以使用


## 实例1
```
$ route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         172.25.64.1     0.0.0.0         UG    0      0        0 eth0
172.25.64.0     0.0.0.0         255.255.240.0   U     0      0        0 eth0
```

第二行表示主机所在网络的地址为 **172.25.64.0**，若数据传送目标是在本局域网内通信，则可以直接通过 eth0 转发数据包；
第一行表示传送的目标是访问 Internet，则由接口 eth0 将数据包发送到网关 **172.25.64.1**。

其中 Flags 标志说明：
- U  Up表示此路由当前为启动状态
- H  Host表示此网关为一个主机
- G  Gateway 表示网关是一个路由器
- R  Reinstate Route，使用动态路由重新初始化的路由
- D  Dynamically 此路由是动态性地写入
- M  Modified 此路由是由路由守护程序或者导向器动态修改
- !  表示此路由当前为关闭状态



## 路由工作原理
路由：数据从源主机到目标主机的转发过程（路径）。

路由器和交换机的区别：
- 数据在同一网段的转发用交换机。
- 数据在不同网段的转发用路由器。

路由器：能够将数据包转移到正确目的地，并在转交过程中选择最佳路径的设备。


## 原理
根据路由表查找接口进行数据转发，相邻的路由器接收到数据包并且查看数据包的目标地址，再转发到网络接口和对应的主机。

## 路由表的形成
路由表是路由器中维护的路由条目的集合，路由器根据路由表做路径选择。
路由表中有直连网段和非直连网段两种。

直连网段：路由器上配置了接口的 IP 地址，并且状态是 Up 状态，由此产生直连路由。
非直连网段：没有跟路由器直接连接的网段，就是非直连网段。

对于非直连网段，需要静态路由和动态路由，将网段添加到路由表中。


## 静态路由和默认路由
- 静态路由：由管理员手工配置，优点在于稳定且可以对路由的行为进行控制，是单向的，缺乏灵活性。
- 默认路由：当路由器在路由表中找不到目标网络的路由条目时，路由器把请求转发到默认路由接口。

当路由表中存在静态路由和默认路由的时候，静态路由优先级最高，匹配上了会立刻转发；如果没有匹配上静态路由，则按照默认路由进行转发。



## 命令行查看路由表

    route -n
    netstat -rn
    ip route show dev <interface>

## 设置网络接口

> ifconfig <interface> <Ip address> netmask <netmask>

    #eg
    ifconfig eth0 192.168.1.100 netmask 255.255.255.0

ifconfig 命令已经被新的 ip 命令所替代，建议在新的系统中使用 ip 命令来管理网络接口。

    #新增一个IP地址
    ip addr add 192.168.1.100/24 dev eth0
    #查询
    ip addr show eth0
    #删除
    ip addr del 192.168.1.100/24 dev eth0


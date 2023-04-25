---
title: "tcpdump 抓包"
linkTitle: "tcpdump 抓包"
weight: 150
description: >
  tcpdump 抓包
---

{{% pageinfo %}}
 Tcpdump 是一个网络数据包截获分析工具。支持针对网络层、协议、主机、网络或者端口的过滤。并提供and or not 等逻辑语句。
{{% /pageinfo %}}



```
tcpdump tcp -i eth1 -t -s 0 -c 100 and dst port ! 22 and src net 192.168.1.0/24 -w ./tag.cap
```

解释如下：
- tcp ：过滤数据包的类型，放到第一个参数位置， icmp arp tcp udp 等
- -i eth1 ：只抓经过接口 eth1 的包
- -t ：不显示时间戳
- -s 0 ：抓完整的数据包，因为抓取数据包时默认的长度为68字节
- -c 100 ：只抓取 100 个数据包
- dst port ! 22 ： 不抓取目标端口是22的数据包
- src net 192.168.1.0/24 ： 数据包的源网络地址是 192.168.1.0
- -w ./tag.cap 保存成 cap 文件，方便 wireshark 分析
---
title: "内核"
linkTitle: "内核"
weight: 8
description: >
  
---

<!-- {{% pageinfo %}}

{{% /pageinfo %}} -->


内核算是一个程序员的基本功，更好的理解底层技术。在处理问题时，可以挖掘到问题本质，而不只是停留在技术表面。
特别是在容器领域设计到 cgroups / namespace / unionfs 等基础技术，需要更加深入学习、掌握。

内核是操作系统的核心，其主要功能有：
- 响应中断，执行中断服务程序
- 管理多个进程，调度和分享处理器时间
- 管理进程地址空间和内存管理
- 网络和进程间通信等系统服务程序


内核的活动范围：
- 运行于用户空间，习性用户进程
- 运行于内核空间，处理进程上下文，代表某个特定进程的执行
- 运行于内核空间，处理中断上下文，与进程无关，处理某个特定的中断


> 官网 https://kernel.org/

本系列围绕在 “进程管理”，“内存管理”，“IO栈”，“网络栈”四大脉络，总结 Linux Kernel 的一些常识知识。


## Kernel 版本
Linux内核根据版本具有不同的支持级别，长期支持版本（LTS），超长期支持（SLTS）。

![kernel-version](/images/linux/kernel-version.png)

```
# 查询内核版本
$ uname -a
Linux xxx 5.10.16.3-microsoft-standard-WSL2 #1 SMP Fri Apr 2 22:23:49 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux  

$ uname -a
Linux xxx 5.4.0-105-generic #119-Ubuntu SMP Mon Mar 7 18:49:24 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux

$ uname -a
Linux xxx 5.15.81-flatcar #1 SMP Tue Dec 6 22:42:27 -00 2022 x86_64 Intel(R) Xeon(R) CPU E5-2680 v4 @ 2.40GHz GenuineIntel GNU/Linux

# 查询系统版本
$ cat /etc/issue
Ubuntu 20.04.4 LTS \n \l

```

内核版本说明：
1. 前三组数字为：主版本号 + 次版本号 + 修订版本号。
2. SMP ： 对称多处理机，表示内核支持多核、多处理器。
3. x86_64 ： 采用64位的cpu


## 抽象级别和层次
最底层是硬件系统，包括cpu和ram，此外硬盘和网络接口也是硬件系统的一部分。

硬件系统之上是内核，是操作系统的核心。内核是运行在内存中的软件，向cpu发送指令。内核管理硬件系统，是硬件系统和应用程序之间进行通信的接口。

进程是计算机中运行的程序，由内核统一管理，组成了最顶层，称为 **用户空间**。

![basic-component](/images/linux/basic-component.png)

内核和用户进程之间最主要的区别是：内核在**内核模式**下运行，而用户进程是在**用户模式**中运行。在内核模式中运行的代码可以不受限的访问cpu和内存，这种模式功能强大，因为内核进程可以让整个系统崩溃。只有内核可以访问的空间称为**内核空间**。
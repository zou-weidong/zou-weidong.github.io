---
title: "虚拟机"
linkTitle: "虚拟机"
weight: 3
description: >
  JVM 虚拟机
---

{{% pageinfo %}}

{{% /pageinfo %}}

## java 内存模型
jmm 体现在以下三个方面：
1. 原子性：保证指令不会受到上下文切换的影响；
2. 可见性：保证指令不会受到cpu缓存的影响；
3. 有序性：保证指令不会受并行优化的影响；


## volaile
解决了可见性和有序性，通过内存屏障实现的。没有解决原子性，即不能通过该关键字实现线程安全。

## 无锁-CAS
compare and swap

为变量赋值时，从内存中读取到的值为v,获取到要交换的新值n，执行 compareAndSwap() 方法时，比较v和当前内存中的值是否一致，如果一致将n和v交换，如果不一致，则自旋重试。

casd底层是cpu层面的，即时不适用同步锁也会保证元资信的操作。


## 线程池

- corePoolSize    核心线程数
- maximumPoolSize 最大线程数
- keepAliveTime   救急线程（max-core）空闲时间
- unit            救急线程（max-core）空闲时间单位
- workQueue       阻塞队列
- threadFactry    创建线程的工厂，主要定义线程名
- handler         拒绝策略

### 线程池的状态
- Running  正常接收任务，正常处理任务
- Shutdown 不接收任务，会执行完正在执行的任务，处理阻塞队列中的任务
- stop  不接收任务，中断正在执行的任务，放弃处理阻塞队列中的任务
- Tidying 即将进入终结
- Termitted 终结状态


### 线程池的主要流程
1. 创建线程池后，线程池的状态是 Running，该状态下才能有以下步骤。
2. 提交任务，线程池创建线程去处理。
3. 当线程池的工作线程数达到 corePoolSize 时，继续提交任务会进入阻塞队列。
4. 当阻塞队列队列装满时，继续提交任务，会创建救急线程来处理。
5. 当线程池中的线程数达到max时，会执行拒绝策略。
6. 当线程取任务的时间达到 keepAliveTime 时还没有取到任务，工作线程数大于 corePoolSize 时，会回收该线程。


### 拒绝策略
1. 调用者抛出 RejectedExecutionException（默认策略）。
2. 让调用者运行任务。
3. 丢弃此次任务。
4. 丢弃阻塞队列中最早的任务，加入该任务。
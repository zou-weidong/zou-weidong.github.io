---
title: "Namespace"
linkTitle: "Namespace"
weight: 1
description: >
  Namespace
---

{{% pageinfo %}}
目前的容器技术、虚拟化技术都能做到资源层面的隔离和限制。

对于容器技术而言，实现资源层面上的限制和隔离，依赖于 Linux 内核所提供的 **cgroup** 和 **namespace** 技术。

- cgroup的主要作用：管理资源的分配、限制。
- namespace的主要作用：封装抽象，限制，隔离，使命名空间内的进程看起来拥有全局资源。
{{% /pageinfo %}}


## Namespace 是什么？
Namespace 是 Linux 内核的一项特性，可以对内核资源进行分区，使一组进程可以看到一组资源；而另一组进程可以看到另一组不同的资源。
该功能的原理时一组资源和进程使用相同的 namespace ，但是这些 namespace 实际上引用的是不同的资源。

Linux 默认提供了多种 namespace，用于对多种不同资源进行隔离。

在之前，单独使用 namespace 的场景比较有限，但是 namespace 却是容器化技术的基石。


## 发展历程
Namespace 开始进入 Linux Kernel 的版本是 2.4.x，才开始实现每个进程的 namespace。
Linux 3.8 中完全实现了 User Namespace 的相关功能集成到内核。这样 Docker 以及其他容器技术所使用的到的 namespace 相关功能就基本实现了。

## Namespace 类型

1. Cgroup - CLONE_NEWCGROUP  控制 Cgroup root directory cgroup 根目录。
2. IPC - CLONE_NEWIPC        控制 System V IPC POSIX message qucucs 信号量，消息队列
3. Nework - CLONE_NEWNET     控制 Network devics , stacks ,ports ,etc 网络设备，协议栈，端口等
4. Mount - CLONE_NEWNS       控制 Mount points 挂载点
5. PID - CLONE_NEWPID        控制 Process IDs 进程号
6. Time - CLONE_NEWTIME      控制时钟
7. User - CLONE_NEWUSER      用户和组ID
8. UTS  - CLONE_NEWUTS       系统主机名和NIS主机名（也称域名）

### 1. Cgroup namespaces
Cgroup namespace 是进程的 cgroups 的虚拟化视图，通过 **/proc/[pid]/cgroup** 和 **/proc/[pid]/mountinfo** 展示。

```
az6c@WXMIS037:/proc/6184$ ps -ef|grep nginx
root      6184  6163  0 10:20 ?        00:00:00 nginx: master process nginx -g daemon off;
...
az6c@WXMIS037:/proc/6184$ cat cgroup
13:rdma:/
12:pids:/docker/828161c00ee69e3c037a090b8b9d295eea22fe5e81b1a18c85962c5477edd7fd
11:hugetlb:/docker/828161c00ee69e3c037a090b8b9d295eea22fe5e81b1a18c85962c5477edd7fd
10:net_prio:/docker/828161c00ee69e3c037a090b8b9d295eea22fe5e81b1a18c85962c5477edd7fd
9:perf_event:/docker/828161c00ee69e3c037a090b8b9d295eea22fe5e81b1a18c85962c5477edd7fd
8:net_cls:/docker/828161c00ee69e3c037a090b8b9d295eea22fe5e81b1a18c85962c5477edd7fd
7:freezer:/docker/828161c00ee69e3c037a090b8b9d295eea22fe5e81b1a18c85962c5477edd7fd
6:devices:/docker/828161c00ee69e3c037a090b8b9d295eea22fe5e81b1a18c85962c5477edd7fd
5:memory:/docker/828161c00ee69e3c037a090b8b9d295eea22fe5e81b1a18c85962c5477edd7fd
4:blkio:/docker/828161c00ee69e3c037a090b8b9d295eea22fe5e81b1a18c85962c5477edd7fd
3:cpuacct:/docker/828161c00ee69e3c037a090b8b9d295eea22fe5e81b1a18c85962c5477edd7fd
2:cpu:/docker/828161c00ee69e3c037a090b8b9d295eea22fe5e81b1a18c85962c5477edd7fd
1:cpuset:/docker/828161c00ee69e3c037a090b8b9d295eea22fe5e81b1a18c85962c5477edd7fd
0::/
az6c@WXMIS037:/proc/6184$ cat mountinfo
157 114 0:50 / / rw,relatime master:3 - overlay overlay rw,lowerdir=/var/lib/docker/overlay2/l/VVZTUGALNCARSF7HMUB445JPUY:/var/lib/docker/overlay2/l/G3PPKI2BXA3PKTPL7VURBQVVKD:/var/lib/docker/overlay2/l/7NVHGK2ITPX2UT2YGDAJFJE2QD:/var/lib/docker/overlay2/l/GZGJV5VQ4T2ZGCCMGFKXOESBQP:/var/lib/docker/overlay2/l/PRL3DYKGN4DW6BAEZZ3B2YMPMZ:/var/lib/docker/overlay2/l/JT7YW37MP5DRFGKGMD3STR64HG:/var/lib/docker/overlay2/l/VVD7Z66THINOLXHB3V3UHUT5K7,upperdir=/var/lib/docker/overlay2/00ba47aabb576aee7cd3ed472f825427050eb44dc909af8dbf667d3619f4f66a/diff,workdir=/var/lib/docker/overlay2/00ba47aabb576aee7cd3ed472f825427050eb44dc909af8dbf667d3619f4f66a/work
158 157 0:53 / /proc rw,nosuid,nodev,noexec,relatime - proc proc rw
159 157 0:54 / /dev rw,nosuid - tmpfs tmpfs rw,size=65536k,mode=755
160 159 0:55 / /dev/pts rw,nosuid,noexec,relatime - devpts devpts rw,gid=5,mode=620,ptmxmode=666
161 157 0:56 / /sys ro,nosuid,nodev,noexec,relatime - sysfs sysfs ro
162 161 0:57 / /sys/fs/cgroup rw,nosuid,nodev,noexec,relatime - tmpfs tmpfs rw,mode=755
163 162 0:33 /docker/828161c00ee69e3c037a090b8b9d295eea22fe5e81b1a18c85962c5477edd7fd /sys/fs/cgroup/cpuset ro,nosuid,nodev,noexec,relatime - cgroup cgroup rw,cpuset
164 162 0:34 /docker/828161c00ee69e3c037a090b8b9d295eea22fe5e81b1a18c85962c5477edd7fd /sys/fs/cgroup/cpu ro,nosuid,nodev,noexec,relatime - cgroup cgroup rw,cpu
165 162 0:35 /docker/828161c00ee69e3c037a090b8b9d295eea22fe5e81b1a18c85962c5477edd7fd /sys/fs/cgroup/cpuacct ro,nosuid,nodev,noexec,relatime - cgroup cgroup rw,cpuacct
166 162 0:36 /docker/828161c00ee69e3c037a090b8b9d295eea22fe5e81b1a18c85962c5477edd7fd /sys/fs/cgroup/blkio ro,nosuid,nodev,noexec,relatime - cgroup cgroup rw,blkio
167 162 0:37 /docker/828161c00ee69e3c037a090b8b9d295eea22fe5e81b1a18c85962c5477edd7fd /sys/fs/cgroup/memory ro,nosuid,nodev,noexec,relatime - cgroup cgroup rw,memory
168 162 0:38 /docker/828161c00ee69e3c037a090b8b9d295eea22fe5e81b1a18c85962c5477edd7fd /sys/fs/cgroup/devices ro,nosuid,nodev,noexec,relatime - cgroup cgroup rw,devices
169 162 0:39 /docker/828161c00ee69e3c037a090b8b9d295eea22fe5e81b1a18c85962c5477edd7fd /sys/fs/cgroup/freezer ro,nosuid,nodev,noexec,relatime - cgroup cgroup rw,freezer
170 162 0:40 /docker/828161c00ee69e3c037a090b8b9d295eea22fe5e81b1a18c85962c5477edd7fd /sys/fs/cgroup/net_cls ro,nosuid,nodev,noexec,relatime - cgroup cgroup rw,net_cls
171 162 0:41 /docker/828161c00ee69e3c037a090b8b9d295eea22fe5e81b1a18c85962c5477edd7fd /sys/fs/cgroup/perf_event ro,nosuid,nodev,noexec,relatime - cgroup cgroup rw,perf_event
172 162 0:42 /docker/828161c00ee69e3c037a090b8b9d295eea22fe5e81b1a18c85962c5477edd7fd /sys/fs/cgroup/net_prio ro,nosuid,nodev,noexec,relatime - cgroup cgroup rw,net_prio
173 162 0:43 /docker/828161c00ee69e3c037a090b8b9d295eea22fe5e81b1a18c85962c5477edd7fd /sys/fs/cgroup/hugetlb ro,nosuid,nodev,noexec,relatime - cgroup cgroup rw,hugetlb
174 162 0:44 /docker/828161c00ee69e3c037a090b8b9d295eea22fe5e81b1a18c85962c5477edd7fd /sys/fs/cgroup/pids ro,nosuid,nodev,noexec,relatime - cgroup cgroup rw,pids
175 162 0:45 / /sys/fs/cgroup/rdma ro,nosuid,nodev,noexec,relatime - cgroup cgroup rw,rdma
176 159 0:52 / /dev/mqueue rw,nosuid,nodev,noexec,relatime - mqueue mqueue rw
177 159 0:58 / /dev/shm rw,nosuid,nodev,noexec,relatime - tmpfs shm rw,size=65536k
178 157 8:16 /var/lib/docker/containers/828161c00ee69e3c037a090b8b9d295eea22fe5e81b1a18c85962c5477edd7fd/resolv.conf /etc/resolv.conf rw,relatime - ext4 /dev/sdb rw,discard,errors=remount-ro,data=ordered
179 157 8:16 /var/lib/docker/containers/828161c00ee69e3c037a090b8b9d295eea22fe5e81b1a18c85962c5477edd7fd/hostname /etc/hostname rw,relatime - ext4 /dev/sdb rw,discard,errors=remount-ro,data=ordered
180 157 8:16 /var/lib/docker/containers/828161c00ee69e3c037a090b8b9d295eea22fe5e81b1a18c85962c5477edd7fd/hosts /etc/hosts rw,relatime - ext4 /dev/sdb rw,discard,errors=remount-ro,data=ordered
121 158 0:53 /bus /proc/bus ro,nosuid,nodev,noexec,relatime - proc proc rw
122 158 0:53 /fs /proc/fs ro,nosuid,nodev,noexec,relatime - proc proc rw
123 158 0:53 /irq /proc/irq ro,nosuid,nodev,noexec,relatime - proc proc rw
124 158 0:53 /sys /proc/sys ro,nosuid,nodev,noexec,relatime - proc proc rw
125 158 0:59 / /proc/acpi ro,relatime - tmpfs tmpfs ro
126 158 0:54 /null /proc/kcore rw,nosuid - tmpfs tmpfs rw,size=65536k,mode=755
127 158 0:54 /null /proc/keys rw,nosuid - tmpfs tmpfs rw,size=65536k,mode=755
128 158 0:54 /null /proc/timer_list rw,nosuid - tmpfs tmpfs rw,size=65536k,mode=755
129 158 0:54 /null /proc/sched_debug rw,nosuid - tmpfs tmpfs rw,size=65536k,mode=755
130 161 0:60 / /sys/firmware ro,relatime - tmpfs tmpfs ro

```

cgroup namespce 提供一系列的隔离支持：
- 防止信息泄漏（容器不应该看到容器外的任何信息）
- 简化容器迁移
- 限制容器进程资源，因为会把 cgroup 文件系统进行挂载，使得容器进程无法获取上层的访问权限。


## Namespace 的主要 API

### clone(2)
系统调用 clone(2) 创建一个新的进程，根据参数中的 CLONE_NEW* 设置，逐个对实现对应的配置功能。当然这个系统调用也实现了一些与 namespace 无关的功能。

### unshare(2)
系统调用 unshare(2)将进程分配至新的 namespace ，同样，也会根据参数中的 CLONE_NEW* 设置调整实现对应的配置功能。

### setns(2)
将进程移动到某一存在的 namespace，这会导致 /proc/[pid]/ns 对应的目录中内容的变更。
进程创建的子进程可以通过调用 unshare(2) 和 setns(2) 来调整所属的 namespace。


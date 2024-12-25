---
title: "Overlay2"
linkTitle: "Overlay2"
weight: 130
description: >
  Overlay2
---

{{% pageinfo %}}
Docker 的存储驱动 **Overlay2**，可以通过 **storage-driver** 参数进行指定，也可以在 **/etc/docker/daemon.json** 文件中通过 **storage-driver** 字段进行设置。
{{% /pageinfo %}}



## 存储驱动的作用
Docker 将容器镜像做了分层存储，每个层相当于包含着一条 Dockerfile 的指令。而这些层在磁盘上存储方式，已经在启动容器时，如何组织这些层，并提供可写层，便是存储驱动的主要作用。

不同的存储驱动实现不同，性能也有差异，同时使用不同的存储驱动会导致占用的磁盘空间有所不同。


## 驱动
- overlay2
- fuse-overlayfs
- btrfs
- zfs
- aufs
- overlay
- devicemapper
- vfs


## OverlayFS
**overlay2** 是 **overlay** 的升级版，这两个存储驱动所用的都是 **OverlayFS**。它的出现是为了解决 overlay 存储驱动 inode 耗尽问问题。


启动一个容器后，查看 mount 挂载

```
$ mount |grep overlay  
overlay on /var/lib/docker/overlay2/00ba47aabb576aee7cd3ed472f825427050eb44dc909af8dbf667d3619f4f66a/merged type overlay 
(rw,relatime,
lowerdir=/var/lib/docker/overlay2/l/VVZTUGALNCARSF7HMUB445JPUY:/var/lib/docker/overlay2/l/G3PPKI2BXA3PKTPL7VURBQVVKD:/var/lib/docker/overlay2/l/7NVHGK2ITPX2UT2YGDAJFJE2QD:/var/lib/docker/overlay2/l/GZGJV5VQ4T2ZGCCMGFKXOESBQP:/var/lib/docker/overlay2/l/PRL3DYKGN4DW6BAEZZ3B2YMPMZ:/var/lib/docker/overlay2/l/JT7YW37MP5DRFGKGMD3STR64HG:/var/lib/docker/overlay2/l/VVD7Z66THINOLXHB3V3UHUT5K7,
upperdir=/var/lib/docker/overlay2/00ba47aabb576aee7cd3ed472f825427050eb44dc909af8dbf667d3619f4f66a/diff,
workdir=/var/lib/docker/overlay2/00ba47aabb576aee7cd3ed472f825427050eb44dc909af8dbf667d3619f4f66a/work)
```

- 查看  **merged** 目录，为容器根目录，可以通过写文件测试。
- lowerdir 是 mount 中指定的目录。
- upperdir 
- workdir 

查看上级目录

```
$ ls /var/lib/docker/overlay2/00ba47aabb576aee7cd3ed472f825427050eb44dc909af8dbf667d3619f4f66a
diff  link  lower  merged  work
```

- lower 是基础层，可以包含多个 lowerdir
- diff 是可写层，即挂载时的 upperdir，在容器内变更的文件都在这层存储
- merged 是最终合并的结果，即容器给我们呈现的结果


## Overlay2
经过上面对 Docker 启动容器之后挂载的 OverlayFS 的介绍后， Overlay2 的工作流程应该会比较清楚。

将镜像各层为 lower 基础层，同时增加 diff 这个可写层，通过 OverlayFS 的工作机制，最终将 merged 作为容器内的文件目录展示给用户。

其中对 lower 的深度有硬编码的限制，当前限制为 128 。如果在使用的过程中遇到错误，有可能表示超过了最大深度限制，必须减少层数。

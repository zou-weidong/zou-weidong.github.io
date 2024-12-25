---
title: "docker 四种网络模型"
linkTitle: "docker 四种网络模型"
weight: 4
description: >
  
---

{{% pageinfo %}}
- bridge：   使用 --network bridge 指定，默认使用 docker0 ；
- host：     使用 --network host   指定；
- none：     使用 --network none   指定；
- container：使用 --network container:（name/id）指定。
{{% /pageinfo %}}

## Docker 四种网络模型：

### 1. bridge 模式
Docker的默认网络设置，此模式会为每一个容器分配 Network Namespace，设置ip等，并将一个主机上的 Docker 容器连接到一个虚拟网桥上（docker0）上。

### 2. host 模式
启动容器时使用host模式，就不会获取一个独立的network Namespace，而是和宿主机公用一个 Network Namespace。
容器将不会虚拟出自己的网络，配置IP等，都是使用宿主机的IP和端口。

### 3. none 模式
在 none 模式下，Docker容器拥有自己的 Network Namespace，但是，并不为docker容器进行任何网络配置。
也就是说这个docker容器没有网卡、IP、路由等信息。

### 4. container
这个模式是新创建的容器和已存在的容器共享一个 Network Namespace，而不是和宿主机共享。
新创建的容器不会创建自己相关的网卡，而是和指定的容器共享Ip，端口范围等。
同样，除了网络方面。在文件系统，进程列表等方面还是资源隔离的。


## 常用命令

```shell
docker network -h
docker network create my-bridge -d bridge --subnet=172.21.0.0/16
docker run -d --net my-bridge --name nginx nginx
```



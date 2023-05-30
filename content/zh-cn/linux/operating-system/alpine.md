---
title: "Alpine"
linkTitle: "Alpine"
weight: 1
description: >
  Alpine
---

{{% pageinfo %}}
**Alpine** 操作系统是一个面向安全的轻型Linux发行版。相比其他操作系统，体积非常小。提供自己的包管理工具 apk，可以直接安装各种软件。

**Alpine Docker** 镜像也继承了 Alpine Linux 发行版的优势，体积只有5M左右。作为基础镜像，有许多好处，如下载快，镜像安全性高，占用更少磁盘等。
{{% /pageinfo %}}


## apk 包管理

首先替换默认的安装源，采用国内一些源：
- 中科大：http://mirrors.ustc.edu.cn/alpine/
- 阿里云：https://mirrors.aliyun.com/alpine/
- 清华大学：https://mirror.tuna.tsinghua.edu.cn/alpine/

使用方法：
```shell
sed -i 's/dl-cdn.alpinelinux.org/mirror.tuna.tsinghua.edu.cn/g' /etc/apk/repositories
apk update # 更新索引生效
apk upgrade --no-cache # 安装可用的升级
```

> Alpine中软件安装包名字可能回合其他发行版不同，可以在 [alpine-package](https://pkgs.alpinelinux.org/packages)  网站搜索。


包管理命令:

**查找包:**
```shell
apk search # 查找所有可用包
apk search -v # 带描述
```

**安装包:**
```shell
apk add openssh # 安装
apk add openssh vim # 安装多个
apk add --no-cache vim # 不使用本地镜像源缓存，相当于先执行 update ，再执行add
```

**安装信息:**
```shell
apk info # 已安装的包
apk info -a vim # 显示完整的软件包信息
```

**更新包:**
```shell
apk upgrade # 升级所有软件
apk upgrade openssh # 指定升级
apk upgrade openssh vim # 指定升级多个
```

**删除包:**
```shell
apk del vim # 删除
```

## 服务管理
alpine 没有使用 **systemctl** 进行服务管理，使用 **rc** 系列命令。
> 精简版的 alpine 没有rc系列命令，可以使用 **apk add --no-cache openrc** 安装

- rc-update  增加或者删除服务
- rc-status  状态管理
- rc-service 管理服务的状态
- openrc 管理不同的运行级

```shell
rc-status -a # 列出所有服务
rc-udate add docker boot # 增加服务到系统启动时运行
rc-service networking restart # 重启网络服务

```

## docker-alpine 镜像
大部分官方镜像都支持使用 Alpine 作为基础镜像。


### 制作镜像

```Dockerfile
FROM alpine:3

RUN sed -i 's/dl-cdn.alpinelinux.org/mirror.tuna.tsinghua.edu.cn/g' /etc/apk/repositories

RUN apk update \
    && apk upgrade --no-cache \
    && apk add --no-cache vim
```


## 相关资源

- Alpine 官网：https://www.alpinelinux.org/
- Alpine 官方仓库：https://github.com/alpinelinux
- Alpine 官方镜像：https://hub.docker.com/_/alpine/
- Alpine 官方镜像仓库：https://github.com/gliderlabs/docker-alpine
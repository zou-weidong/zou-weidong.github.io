---
title: "Flatcar Container Linux"
linkTitle: "Flatcar Container Linux"
weight: 2
description: >
  Flatcar Container Linux
---

{{% pageinfo %}}
**Flatcar** 是一种容器优化的操作系统，提供了一个最小的操作系统镜像，其中仅包含运行容器所需要的工具。操作系统通过不可变的文件系统交付，并包括自动院子更新。
{{% /pageinfo %}}


## 安装
支持大多数的云供应商、虚拟化平台和裸机服务器上运行。这里主要介绍几种不同的方式在裸机安装：
- 从 ISO 镜像安装
- 使用 PXE 引导
- 使用 iPXE 启动
- 使用 flatcar-install 安装


### iPXE

**iPXE** 是一个开源网络启动固件。提供了一个完整的 PXE 实施，增强了其他功能，例如：
- 通过 HTTP 从 web 服务器启动
- 从无线网络启动
- 从广域网启动
- 从 Infiniband 网络启动
- 使用脚本控制启动过程
- ...

iPXE 同时支持 UEFI 和 BIOS 平台。


### PXE
**PXE** 是 **Perboot eXecution Environment** 的缩写，意为“预启动执行环境”，这种机制可以使计算机通过网络来引导。现在的绝大多数电脑都可以设置通过 PXE 启动。
PXE 的工作原理是 PXE Client 通过 DHCP 获取 IP，并由 DHCP 服务器告诉客户端启动文件的位置，再通过 TFTP 协议传输引导文件，最终引导电脑启动。


### 使用 flatcar-install 安装
使用 **flatcar-install** 脚本安装.

> fatcar-install -d /dev/sda -i ignition.json

**ignition.json** 文件包括从 Butane Config 生成的用户信息，否则无法登陆 Flatcar 实例。
如果是在 VMware 上安装，传递 -o vmware_raw 以安装特定于 VMware 的镜像。

> flatcar-install -d /dev/sda -i ignition.json -o vmware_raw

### 选择 channel
Flatcar Container Linux根据每个频道的不同时间表自动更新，可以禁用此功能。生产环境使用 stable channel，要确保使用稳定版本，使用以下选项：

> flatca-install -d /dev/sda -C stable

## Butane Configs
默认情况下，没有密码和其他方式登陆到新的 Flatcar Container Linux 系统，配置账户、添加 systemd 单元等最简单的方法是通过 Butane Configs。
使用 Butane 生成 Ignition 配置后，安装脚本将处理 ignition.json 文件。

为用户指定ssh密钥core的yaml文件如下：
```
variant: flatcar
version: 1.0.0
passwd:
  users:
    - name: core
      ssh_authorized_keys:
        - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDGdByTgSVHq.......
```

将其转译为 json 文件：
```
cat c1.yaml |docker run --rm -i quay.io/coreos/butane:latest > ignition.json
```


```
sudo ifconfig eno1 172.xxx.xxx.xxx netmask 255.255.255.0
sudo route add default gw 172.xxx.xxx.1

#... 添加 config.json 文件
sudo -E flatcar-install -C stable -d /dev/sda -i config.json
```

```yaml
passwd:
  users:
    - name: "core"
      ssh_authorized_keys:
        - "ssh-rsa AAAA
    - name: "aaa"
      password_hash: "$6$brlNJCk"
      ssh_authorized_keys:
        - "ssh-rsa AAAAB3NzaC1yc2E"
      groups:
        - "sudo"
        - "docker"
networkd:
  units:
    - name: 00-eno.network
      contents: |
        [Match]
        Name=eno*
 
        [Network]
        Bond=bond0
    - name: 10-bond0.netdev
      contents: |
        [NetDev]
        Name=bond0
        Kind=bond
 
        [Bond]
        Mode=active-backup
        MIIMonitorSec=1
    - name: 20-bond0.network
      contents: |
        [Match]
        Name=bond0
 
        [Network]
        DNS=172.xxx
        DNS=172.xxx
        Address=172.xxx/24
        Gateway=172.16.164.1
systemd:
  units:
    - name: containerd.service
      dropins:
        - name: 10-use-cgroupfs.conf
          contents: |
            [Service]
            Environment=CONTAINERD_CONFIG=/usr/share/containerd/config-cgroupfs.toml   
    - name: settimezone.service
      enabled: true
      contents: |
        [Unit]
        Description=Set the time zone
 
        [Service]
        ExecStart=/usr/bin/timedatectl set-timezone America/Los_Angeles
        RemainAfterExit=yes
        Type=oneshot
 
        [Install]
        WantedBy=multi-user.target  
    - name: systemd-networkd.service
      enable: true
    - name: update-engine.service
      mask: true
    - name: locksmithd.service
      mask: true
 
storage:
  filesystems:
    - name: "OEM"
      mount:
        device: "/dev/disk/by-label/OEM"
        format: "btrfs"
  files:
    - filesystem: "OEM"
      path: "/grub.cfg"
      mode: 0644
      append: true
      contents:
        inline: |
                    set linux_append="$linux_append systemd.unified_cgroup_hierarchy=0 systemd.legacy_systemd_cgroup_controller"
    - path: /etc/flatcar-cgroupv1
      mode: 0444
    - filesystem: "root"
      path:       "/etc/hostname"
      mode:       0644
      contents:
        inline: nodeIp
    - path: /etc/systemd/timesyncd.conf
      filesystem: root
      mode: 0644
      contents:
        inline: |
          [Time]
          NTP=xxx
```
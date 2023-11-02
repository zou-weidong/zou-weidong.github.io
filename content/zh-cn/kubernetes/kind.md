---
title: "kind 快速搭建k8s测试集群"
linkTitle: "kind 快速搭建k8s测试集群"
weight: 277
description: >
    kind 快速搭建k8s测试集群
---

```
sed -i 's/dl-cdn.alpinelinux.org/a.b.com\/artifactory/g' /etc/apk/repositories

docker run -d -e MYSQL_ROOT_PASSWORD=123456 -p 3306:3306  mysql:8.0.31
```


```
kind delete cluster -n test-cluster
# v1.25.11  v1.26.6  v1.17.3  v1.28.0
kind create cluster --name test-cluster --image kindest/node:v1.28.0  --config cluster-config.yaml

```

```
apiVersion: kind.x-k8s.io/v1alpha4
kind: Cluster
nodes:
- role: control-plane
- role: worker
- role: worker

```

```
#cloud-config
passwd:
  users:
    - name: "core"
      ssh_authorized_keys:
        - "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCtcjZh2+cPHF19Tz9wMDvvilnA2LIfHi15x8SQQGP2y0mUa40K0AysV4+03AFTbbAjGqT288HWF470aNNE8xGUHWUPN0wNZTsiawrQ96fiypefdLkWBYumbEMZm4KFl2oibm+3nM3aWeJUjBz7BGpaXqOdHhpDPloTYsJxAll/tq86hW8k/91pLJEWxkbUCHoUpPB/uXVoPg4qp1VyuWgPz0/GfLfOJXQx6MDcKFogjLbVqBFq/4bdZ3Saz0XQ5uxlphxhQjS6K3UiUCzwGPd1K0MOM3AQ1vC6cdLCrr18TYNbZXgkTCrX4IR1hSsFnNmd7Wy9GGHYcZi1pICKS9IR jude.x.zhu@newegg.com"
    - name: "aaa"
      password_hash: "$6$brlNJCkW6ictVVVS$cuFNQVDLA7dU/zyQqOZeyKxoppH8hmQheoCQwnZMUDXVlcKDEJp/E15GJqRCqcZsRYwto3LpSAs1SGNr7K6jn."
      ssh_authorized_keys:
        - "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCtcjZh2+cPHF19Tz9wMDvvilnA2LIfHi15x8SQQGP2y0mUa40K0AysV4+03AFTbbAjGqT288HWF470aNNE8xGUHWUPN0wNZTsiawrQ96fiypefdLkWBYumbEMZm4KFl2oibm+3nM3aWeJUjBz7BGpaXqOdHhpDPloTYsJxAll/tq86hW8k/91pLJEWxkbUCHoUpPB/uXVoPg4qp1VyuWgPz0/GfLfOJXQx6MDcKFogjLbVqBFq/4bdZ3Saz0XQ5uxlphxhQjS6K3UiUCzwGPd1K0MOM3AQ1vC6cdLCrr18TYNbZXgkTCrX4IR1hSsFnNmd7Wy9GGHYcZi1pICKS9IR jude.x.zhu@newegg.com"
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
        DNS=172.16.166.111
        DNS=172.16.166.112
        Address=172.16.164.133/24
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
        inline: e11k8swk13.mercury.corp
    - path: /etc/systemd/timesyncd.conf
      filesystem: root
      mode: 0644
      contents:
        inline: |
          [Time]
          NTP=172.16.72.21 172.16.72.22 172.16.72.23
```


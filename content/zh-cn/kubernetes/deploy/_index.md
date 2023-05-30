---
title: "k8s集群搭建"
linkTitle: "集群搭建"
weight: 100
description: >
  k8s集群搭建
---

{{% pageinfo %}}
利用 **kubeadm** 和 **二进制** 两种方式搭建集群。
{{% /pageinfo %}}

可以搭建单master集群，管理多个node节点。也可以使用多个master节点，管理多个node节点，同时对多个apiserver使用负载均衡。

## 服务器配置
一般测试环境搭建：
- master：2核 4G
- node：  4核 8G 硬盘30G

生产环境：
- master：8核  16G 
- node： 8/16核 32G/64G 

## 搭建方式
- kubeadm是一个k8s部署工具，提供 kubeadm init 和 kubeadm join，用于快速部署 kubernetes 集群。
- 二进制包方式，下载发行版包，手动部署每个组件，组成 Kubernetes 集群。


## kubeadm 部署集群
kubeadm工具能够通过两条指令完成一个 kubernetes 集群的部署。
- 创建一个master节点 **kubeadm init**
- 将 Node 节点加入到当前集群中 **kubeadm join**

https://kubernetes.io/zh-cn/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

### 安装要求
- 操作系统 Centos7.x
- 集群中的机器互通
- 可以访问外网，需要拉取镜像
- 禁止 swap 分区


### 环境准备

master: 192.168.177.130
node1: 192.168.177.131
node2: 192.168.177.132

在每台机器上执行以下命令：
```
# 关闭防火墙
systemctl stop firewalld
systemctl disable firewalld


# 关闭 selinux
# 永久关闭
sed -i 's/enforcing/disabled/' /etc/selinux/config
# 临时关闭
setenforce 0


# 关闭 swap
# 永久关闭
sed -ri 's/.*swap.*/#&/' /etc/fstab
# 临时关闭
swapoff -a

# 规划主机名（各个机器不同执行）
hostnamectl set-hostname k8smaster/k8snode1/k8snode2

# 添加hosts
cat >> /etc/hosts << EOF
192.168.177.130 k8smaster
192.168.177.131 k8snode1
192.168.177.132 k8snode2
EOF

# 将桥接的 Ipv4 流量传递到 iptables 的链
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
# 生效
sysctl --system  


# 时间同步
yum install ntpdata -y
ntpodata time.windows.com

```

### 所有节点安装 Docker/kubeadm/kubelet

**配置docker源**

https://developer.aliyun.com/mirror/?spm=a2c6h.13651104.0.d1002.2cd533174OAYBv

```
# 使用阿里源
cat >/etc/yum.repos.d/docker.repo<<EOF
[docker-ce-edge]
name=Docker CE Edge - \$basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/\$basearch/edge
enabled=1
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg
EOF

```

**安装docker**

```
# yum安装
yum -y install docker-ce

# 查看docker版本
docker --version  

# 启动docker
systemctl enable docker
systemctl start docker

# 配置docker的镜像源
cat >> /etc/docker/daemon.json << EOF
{
  "registry-mirrors": ["https://b9pmyelo.mirror.aliyuncs.com"]
}
EOF

#重启docker
systemctl restart docker

```

**配置k8s源**

```
cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

**安装 kubeadm kubelet kubectl**

```
# 安装kubelet、kubeadm、kubectl，同时指定版本
yum install -y kubelet-1.18.0 kubeadm-1.18.0 kubectl-1.18.0
# 设置开机启动
systemctl enable kubelet
```

### 部署 kubernetes master node
```
kubeadm init --apiserver-advertise-address=192.168.177.130 --image-repository registry.aliyuncs.com/google_containers --kubernetes-version v1.18.0 --service-cidr=10.96.0.0/12  --pod-network-cidr=10.244.0.0/16
```

当出现 **successfully** 字样时表示已经安装成功。最下面有工作节点添加进集群的命令，需要copy。

```
#使用 kubectl 工具
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# 检查，会发现有一个master节点，但是状态是 NotReady
kubectl get nodes

```

### 部署 kubernetes worker node
复制 **kubeadm init ...** 输出的 **kubeadm join** 命令执行。默认 token 的有效期是 24h，当过期之后需要重新创建token。

```
kubeadm token create --print-join-command
```

当我们把两个节点都加入进来后，可以去 master 节点执行下面命令：
```
# 出现三个节点，但是状态都是 NotReady
kubectl get node
```
### 部署 CNI 网络插件

```
# 下载网络插件配置，需要修改镜像
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

# apply 一下创建，之后会发现已经 Ready

# 如果还是有些节点处于 NotReady 状态，可以在master节点删除该节点，然后节点重置之后重新加入。
kubectl delete node k8snode1
#在 k8snode1 节点上进行重置
kubeadm reset 
# 重置完成后加入
kubeadm join 192.168.177.130:6443 --token 8j6ui9.gyr4i156u30y80xf     --discovery-token-ca-cert-hash sha256:eda1380256a62d8733f4bddf926f148e57cf9d1a3a58fb45dd6e80768af5a500

```

## 二进制搭建 1.24

两台机器，vms71和vms72      
系统centos7.4
- vms71为 master
- vms72 为 worker

### 基本设置

识别地址
```
cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.26.71 vms71.rhce.cc vms71
192.168.26.72 vms72.rhce.cc vms72
```

关闭swap分区 
```
swapoff -a ; sed -i '/fstab/d' /etc/fstab 
```

更新 yum 源
```
rm -rf /etc/yum.repos.d/*  ; wget ftp://ftp.rhce.cc/k8s/* -P /etc/yum.repos.d/ #一般更新为阿里源
yum clean all 
```

安装 containerd
 ```
yum install  containerd.io cri-tools  -y
crictl config runtime-endpoint unix:///var/run/containerd/containerd.sock
```

先生成配置文件/etc/containerd/config.toml
```
containerd config default > /etc/containerd/config.toml
```

编辑配置文件，改成：

```
  [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
          endpoint = ["https://frz7i079.mirror.aliyuncs.com"]

# 修改 sandbox_image
[plugins."io.containerd.grpc.v1.cri"]
# enable SELinux labeling
enable_selinux = true
sandbox_image = "XX/pause:3.6"


# 根据cgroup版本配置是否为true
SystemdCgroup = false
```


在所有节点重启 containerd ，并设置开机自动启动
```
systemctl enable containerd 
systemctl restart containerd
```

在所有机器上执行命令，目的是系统重启时能自动加载。
```
cat > /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF
```


执行以下内核命令
```
cat <<EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

sysctl -p /etc/sysctl.d/k8s.conf
```


安装 nerdctl（containerd cli,可选）
https://github.com/containerd/nerdctl/releases
https://github.com/containernetworking/plugins/releases


修改环境配置文件
```
vim /etc/profile

#添加
source <(nerdctl completion bash)
export CONTAINERD_NAMESPACE=k8s.io


# 生效
source /etc/profile
```


安装 cfssl 工具，放在 /usr/local/bin 中。

```
# ls
cfssl-certinfo_linux-amd64  cfssljson_linux-amd64  cfssl_linux-amd64

#for i in *;do n=${i%_*};mv $i $n;done;chmod +x *

# ls
cfssl  cfssl-certinfo  cfssljson

# 移动到 /usr/local/bin 文件夹

```

### 配置 k8s 各组件所需要的证书

![tls](/images/k8s/tls.png)


操作在一个机器上操作，然后将生成的正式拷贝到 /etc/kubernetes/pki 里。

```
mkdir -p /etc/kubernetes/pki
```

#### 搭建CA
```
# cfssl print-defaults config > ca-config.json
# cat ca-config.json

{
    "signing": {
        "default": {
            "expiry": "87600h"
        },
        "profiles": {
            "www": {
                "expiry": "87600h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "server auth",
                    "client auth"
                ]
            }
        }
    }
}

# cfssl print-defaults csr > ca-csr.json
# cat ca-csr.json

{
    "CN": "kubernetes",
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "Shanxi",
            "L": "xian",
            "O": "kubernetes",
            "OU": "system"
        }
    ]
}

# cfssl gencert -initca ca-csr.json |cfssljson  -bare ca
# ls

ca-config.json  ca.csr  ca-csr.json  ca-key.pem  ca.pem

```

#### front-proxy-ca
创建聚合层(Aggregation Layer)所需要的证书，如果环境里没有配置聚合层，这步和下步创建证书的步骤是不需要的。



生成 front-proxy-ca 自签名证书
```
cp ca-config.json front-proxy-ca-config.json
cp ca-csr.json front-proxy-ca-csr.json
cfssl gencert -initca front-proxy-ca-csr.json | cfssljson -bare front-proxy-ca
```

聚合层客户端证书
```
cfssl print-defaults csr > front-proxy-client-csr.json

{
  "CN": "front-proxy-client",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "Shanxi",
      "L": "XIan",
      "O": "kubernetes",
      "OU": "system"
    }
  ]
}

cfssl gencert -ca=front-proxy-ca.pem -ca-key=front-proxy-ca-key.pem -config=front-proxy-ca-config.json -profile=www front-proxy-client-csr.json | cfssljson  -bare front-proxy-client
```

#### etcd 证书

etcd需要跟apiserver进行双向tls认证，etcd服务器端需要证书和私钥，先创建etcd证书请求文件。


先创建etcd服务端证书
```
cfssl print-defaults csr > etcd-csr.json

{
  "CN": "etcd",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
   "hosts": [
     "127.0.0.1",
     "192.168.26.71",
     "xxx"
    ],
  "names": [
    {
      "C": "CN",
      "ST": "Shanxi",
      "L": "Xian",
      "O": "kubernetes",
      "OU": "system"
    }
  ]
}

# 颁发 etcd 服务器端所需要的证书
cfssl gencert -ca-ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=www etcd-csr.json | cfssljson -bare etcd

ls etcd*.pem
etcd-key.pem etcd.pem
```

再创建 etcd客户端证书，当 apiserver向 etcd 连接时向etcd出示的证书

```
cfssl print-defaults csr > etcd-client-csr.json
cat etcd-client-csr.json

{
  "CN": "etcd",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
   "hosts": [
     "127.0.0.1",
     "192.168.26.71",
     "xxx"
    ],
  "names": [
    {
      "C": "CN",
      "ST": "Shanxi",
      "L": "Xian",
      "O": "kubernetes",
      "OU": "system"
    }
  ]
}


cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=www etcd-client-csr.json | cfssljson  -bare etcd-client
ls etcd-client*.pem

etcd-clinet-key.pem  etcd-client.pem
```

#### Apiserver 证书
k8s其他组件要跟 apiserver进行双向tls认证，所以apiserver需要自己的证书，下面生成apiserver所需要的证书请求文件。

```
cfssl print-defaults csr > apiserver-csr.json

{
  "CN": "kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
   "hosts": [
     "127.0.0.1",
     "192.168.26.71", # master节点的Ip填上
     "192.168.26.72",
     "10.96.0.1",  # service网络的第一个ip
     "kubernetes",
     "kubernetes.default",
     "kubernetes.default.svc",
     "kubernetes.default.svc.cluster",
     "kubernetes.default.svc.cluster.local"
    ],
  "names": [
    {
      "C": "CN",
      "ST": "Shanxi",
      "L": "Xian",
      "O": "kubernetes",
      "OU": "system"
    }
  ]
}

cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=www apiserver-csr.json | cfssljson -bare apiserver
ls apiserver*.pem 

apiserver-key.pem  apiserver.pem
```

####  controller-manager

controller-manager 需要跟apiserver 进行 mtls 认证，生成证书申请请求文件

```
cfssl print-defaults csr > controller-manager-csr.json

{
  "CN": "system:kube-controller-manager",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
   "hosts": [
     "127.0.0.1",
     "192.168.26.71"  # 所有 节点ip
    ],
  "names": [
    {
      "C": "CN",
      "ST": "Shanxi",
      "L": "Xian",
      "O": "system:kube-controller-manager",
      "OU": "system"
    }
  ]
}


cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=www controller-manager-csr.json | cfssljson -bare controller-manager
ls -1 control*.pem

controller-manager-key.pem  controller-manager.pem
```

schedule 也用相同的方式，kube-proxy 也用相同的方式（没有host）

#### admin 用到的证书

```
cfssl print-defaults csr > admin-csr.json
cat admin-csr.json

{
  "CN": "admin",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "Jiangsu",
      "L": "Xuzhou",
      "O": "system:masters",
      "OU": "system"
    }
  ]
}

cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=www admin-csr.json | cfssljson -bare admin

```

把所有证书都拷贝到 /etc/kubernetes/pki 里。



```
$ ls /etc/kubernetes/pki
admin-key.pem          ca-key.pem             etcd-client-key.pem  front-proxy-ca-key.pem  proxy-key.pem         admin.pem       ca.pem               etcd-client.pem         front-proxy-ca.pem      proxy.pem
apiserver-key.pem  controller-manager-key.pem  etcd-key.pem         front-proxy-client-key.pem  scheduler-key.pem
apiserver.pem      controller-manager.pem      etcd.pem             front-proxy-client.pem      scheduler.pem
```


#### etcd 安装

> 下载地址 https://github.com/etcd-io/etcd/releases/download/v3.5.4/etcd-v3.5.4-linux-amd64.tar.gz
将下载后的可执行文件拷贝到 **/usr/local/bin** 目录下。

创建两个目录
```
mkdir /etc/etcd /var/lib/etcd
```

配置文件
```
cat /etc/etcd/etcd.conf

#[Member]
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_PEER_URLS="https://192.168.26.71:2380"
ETCD_LISTEN_CLIENT_URLS="https://192.168.26.71:2379,http://127.0.0.1:2379"
ETCD_NAME="etcd1"
#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://192.168.26.71:2380"
ETCD_ADVERTISE_CLIENT_URLS="https://192.168.26.71:2379"
ETCD_INITIAL_CLUSTER="etcd1=https://192.168.26.71:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_INITIAL_CLUSTER_STATE="new"
```

创建etcd的启动脚本。 /usr/lib/systemd/system/etcd.service
```
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target

[Service]
Type=notify
EnvironmentFile=-/etc/etcd/etcd.conf
WorkingDirectory=/var/lib/etcd/
ExecStart=/usr/local/bin/etcd \
  --cert-file=/etc/kubernetes/pki/etcd.pem \
  --key-file=/etc/kubernetes/pki/etcd-key.pem \
  --trusted-ca-file=/etc/kubernetes/pki/ca.pem \
  --peer-cert-file=/etc/kubernetes/pki/etcd.pem \
  --peer-key-file=/etc/kubernetes/pki/etcd-key.pem \
  --peer-trusted-ca-file=/etc/kubernetes/pki/ca.pem \
  --peer-client-cert-auth \
  --client-cert-auth
Restart=on-failure
RestartSec=5
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

启动etcd并设置开机自启

```
systemctl enable etcd --now
systemctl is-actiive etcd
```

#### 配置 kubernetes 各个组件

下载对应的二进制包，解压到对应的目录。如：/usr/bin/

后面配置 kubelet 的 bootstrap 认证，即 kubelet 启动时自动创建 csr 请求，这里需要在 apiserver 上开启 token 的认证，所以现在master上生成一个随机值作为token。

```
openssl rand -hex 10
xxxxxxxxx
```

将token导入到文件里，这里写入到 /etc/kubernetes/token.csv，格式为 token,user,uid,groups
```
xxxxxxxxx,kubelet-bootstrap,10001,"system:node-bootstrapper"
```

创建 **apiserver** 启动脚本:


> cat /usr/lib/systemd/system/kube-apiserver.service

```
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes
After=network.target

[Service]
ExecStart=/usr/bin/kube-apiserver \
      --v=2  \
      --logtostderr=true  \
      --allow-privileged=true  \
      --bind-address=192.168.26.71  \
      --secure-port=6443 \
      --token-auth-file=/etc/kubernetes/token.csv  \
      --advertise-address=192.168.26.71 \
      --service-cluster-ip-range=10.96.0.0/16  \
      --service-node-port-range=30000-60000  \
      --etcd-servers=https://192.168.26.71:2379 \
      --etcd-cafile=/etc/kubernetes/pki/ca.pem  \
      --etcd-certfile=/etc/kubernetes/pki/etcd.pem  \
      --etcd-keyfile=/etc/kubernetes/pki/etcd-key.pem  \
      --client-ca-file=/etc/kubernetes/pki/ca.pem  \
      --tls-cert-file=/etc/kubernetes/pki/apiserver.pem  \
      --tls-private-key-file=/etc/kubernetes/pki/apiserver-key.pem  \
      --kubelet-client-certificate=/etc/kubernetes/pki/apiserver.pem  \
      --kubelet-client-key=/etc/kubernetes/pki/apiserver-key.pem  \
      --service-account-key-file=/etc/kubernetes/pki/ca-key.pem  \
      --service-account-signing-key-file=/etc/kubernetes/pki/ca-key.pem  \
      --service-account-issuer=https://kubernetes.default.svc.cluster.local \
      --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname  \
      --enable-admission-plugins=NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,DefaultTolerationSeconds,NodeRestriction,ResourceQuota  \
      --authorization-mode=Node,RBAC  \
      --enable-bootstrap-token-auth=true
      #--requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.pem  \
      #--proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.pem  \
      #--proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client-key.pem  \
      #--requestheader-allowed-names=aggregator  \
      #--requestheader-group-headers=X-Remote-Group  \
      #--requestheader-extra-headers-prefix=X-Remote-Extra-  \
      #--requestheader-username-headers=X-Remote-User  

Restart=on-failure
RestartSec=10s
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
```

几个选项作用：
- --v 日志等级
- --enable-bootstrap-token-auth ：启用 tls bootstrap 机制
- --token-auth-file ：bootstrap token 文件


启动 **apiserver**
```
systemctl start kube-apiserver
systemctl enable kube-apiserver 
```

配置 **controller-manager**

> cat /usr/lib/systemd/system/kube-controller-manager.service
```
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes
After=network.target

[Service]
ExecStart=/usr/bin/kube-controller-manager \
      --v=2 \
      --logtostderr=true \
      --bind-address=127.0.0.1 \
      --root-ca-file=/etc/kubernetes/pki/ca.pem \
      --cluster-signing-cert-file=/etc/kubernetes/pki/ca.pem \
      --cluster-signing-key-file=/etc/kubernetes/pki/ca-key.pem \
      --service-account-private-key-file=/etc/kubernetes/pki/ca-key.pem \
      --kubeconfig=/etc/kubernetes/kube-controller-manager.kubeconfig \
      --leader-elect=true \
      --use-service-account-credentials=true \
      --node-monitor-grace-period=40s \
      --node-monitor-period=5s \
      --pod-eviction-timeout=2m0s \
      --controllers=*,bootstrapsigner,tokencleaner \
      --allocate-node-cidrs=true \
      --cluster-cidr=10.244.0.0/16 \
      --node-cidr-mask-size=24
Restart=always
RestartSec=10s

[Install]
WantedBy=multi-user.target
```


创建 controller-manager 所需要的 kubeconfig 文件 -- **kube-controller-manager.kubeconfig**

controller-manager 和 apierver 之间的认证是通过 kubeconfig 的方式来认证的，即 controller-manager 的私钥，公钥，ca的证书放在一个 kubeconfig 文件里。
下面创建 controller-manager 所用的 kubeconfig 文件 **kube-controller-manager.kubeconfig**

```
cd /etc/kubernetes/pki/

# 设置集群信息
kubectl config set-cluster kubernetes --certificate-authority=ca.pem --embed-certs=true --server=https://192.168.26.71:6443 --kubeconfig=kube-controller-manager.kubeconfig
# 设置用户信息
kubectl config set-credentials system:kube-controller-manager --client-certificate=controller-manager.pem --client-key=controller-manager-key.pem --embed-certs=true --kubeconfig=kube-controller-manager.kubeconfig
# 设置上下文信息
kubectl config set-context system:kube-controller-manager --cluster=kubernetes --user=system:kube-controller-manager --kubeconfig=kube-controller-manager.kubeconfig
# 设置默认的上下文
kubectl config use-context system:kube-controller-manager --kubeconfig=kube-controller-manager.kubeconfig
#放到合适的地方
mv kube-controller-manager.kubeconfig /etc/kubernetes/

```

启动和开机自启

```
systemctl deamon-reload
systemctl enable kube-controller-manager --now
systemctl start kube-controller-manager
```

配置和启动 scheduler

cat /usr/lib/systemd/system/kube-scheduler.service
```
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes
After=network.target

[Service]
ExecStart=/usr/bin/kube-scheduler \
      --v=2 \
      --logtostderr=true \
      --bind-address=127.0.0.1 \
      --leader-elect=true \
      --kubeconfig=/etc/kubernetes/kube-scheduler.kubeconfig

Restart=always
RestartSec=10s

[Install]
WantedBy=multi-user.target
```

创建scheduler 的 kubeconfig 和启动参考上文。

最后创建admin能用的kubeconfig
- 注意用户是admin，其他都一样，参考上文。


**kubelet**
为用户 kubelet-bootstrap 授权，允许 kubelet tls bootstrap 创建 CSR 请求。
```
kubectl create clusterrolebinding kubelet-bootstrap1 --clusterrole=system:node-bootstrapper --user=kubelet-bootstrap
```

把 system:certificates.k8s.io:certificatesigningrequests:nodeclient 授权给 kubelet-bootstrap ，目的是为了实现对 csr 的自动审批。
```
kubectl create clusterrolebinding kubelet-bootstrap2 --clusterrole=system:certificates.k8s.io:certificatesigningrequests:nodeclient --user=kubelet-bootstrap
```

创建 kubelet bootstap 的 kubeconfig 文件。
- 用户名是 kube-bootstrap
- 用指定的token  --token 


创建启动脚本 ，后面会把 kubelet 颁发的证书放在 /var/lib/kubelet/pki 里，所以先把此目录创建出来。
```
mkdir -p /var/lib/kubelet/pki /var/log/kubernetes
```

cat /usr/lib/systemd/system/kubelet.service
```
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=containerd.service
Requires=containerd.service

[Service]
WorkingDirectory=/var/lib/kubelet
ExecStart=/usr/bin/kubelet \
  --bootstrap-kubeconfig=/etc/kubernetes/kubelet-bootstrap.conf  \
  --cert-dir=/var/lib/kubelet/pki \
  --kubeconfig=/etc/kubernetes/kubelet.kubeconfig \
  --config=/etc/kubernetes/kubelet-config.yaml \

  --pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.7 \
  --container-runtime=remote \
  --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock \
  
  --runtime-request-timeout=15m \
  --alsologtostderr=true \
  --logtostderr=false \
  --log-dir=/var/log/kubernetes \
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```


创建 kubelet 用的配置文件 **kubelet-config.yaml**

```
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
address: 0.0.0.0
port: 10250 
readOnlyPort: 10255
authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 0s
    enabled: true
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.pem
authorization:
  mode: Webhook
  webhook:
    cacheAuthorizedTTL: 0s
    cacheUnauthorizedTTL: 0s
cgroupDriver: systemd
clusterDNS:
- 10.96.0.10
clusterDomain: cluster.local
cpuManagerReconcilePeriod: 0s
evictionPressureTransitionPeriod: 0s
fileCheckFrequency: 0s
healthzBindAddress: 127.0.0.1
httpCheckFrequency: 0s
imageMinimumGCAge: 0s
```

这里指定的 cluster dns 的ip是 10.96.0.10

最后将kubelet 能用到的文件同步到所有的worker节点上，启动kubelet并设置开机自启。


查看 csr已经有申请请求自动审批了。



安装 **kube-proxy**

kube-proxy.yaml
```
apiVersion: kubeproxy.config.k8s.io/v1alpha1
bindAddress: 0.0.0.0
clientConnection:
  kubeconfig: /etc/kubernetes/kube-proxy.kubeconfig
clusterCIDR: 10.244.0.0/16 
kind: KubeProxyConfiguration
metricsBindAddress: 0.0.0.0:10249
mode: "ipvs"
```

再创建一个 kubeconfig 同上

```
[Unit]
Description=Kubernetes Kube-Proxy Server
Documentation=https://github.com/kubernetes/kubernetes
After=network.target

[Service]
WorkingDirectory=/var/lib/kube-proxy
ExecStart=/usr/bin/kube-proxy \
  --config=/etc/kubernetes/kube-proxy.yaml \
  --alsologtostderr=true \
  --logtostderr=false \
  --log-dir=/var/log/kubernetes \
  --v=2
Restart=on-failure
RestartSec=5
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```


rbac
```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: system:kubernetes-to-kubelet
rules:
  - apiGroups:
      - ""
    resources:
      - nodes/proxy
      - nodes/stats
      - nodes/log
      - nodes/spec
      - nodes/metrics
    verbs:
      - "*"
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: system:kubernetes
  namespace: ""
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:kubernetes-to-kubelet
subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: User
    name: kubernetes
```

安装网络插件，部署coredns，最后测试.


https://www.rhce.cc/3684.html
---
title: "cfssl 使用"
linkTitle: "cfssl 使用"
weight: 229
description: >
    启用加密，防止流量拦截和中间人攻击。
---

## 1. 下载 cfssl
在github上的 [cfssl源码和下载地址](https://github.com/cloudflare/cfssl) 。

```shell
mkdir ~/bin
curl -s -L -o ~/bin/cfssl https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
curl -s -L -o ~/bin/cfssljson https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
curl -s -L -o ~/bin/cfssl-certinfo https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64
chmod +x ~/bin/{cfssl,cfssljson,cfssl-certinfo}
# 临时生效
export PATH=$PATH:~/bin
# 写到文件中  export PATH=$PATH:/root/bin
source /etc/profile
```

## 2. 初始化CA证书
```
mkdir ~/cfssl
cd ~/cfssl
cfssl print-defaults config > ca-config.json
cfssl print-defaults csr > ca-csr.json
```

以生成 kubernetes 或者 etcd 自签证书为例:

### 配置 CA 信息

一般来说，profiles 节点有多个配置，**server端**证书带有 server auth ，**client端**证书带有 client auth ，**peer 端**的证书需要 server auth 和 client auth。时间默认值为 8760h

```json
"profiles": {
    "server": {
        "expiry": "8760h",
        "usages": [
            "signing",
            "key encipherment",
            "server auth"
        ]
    },
    "client": {
        "expiry": "8760h",
        "usages": [
            "signing",
            "key encipherment",
            "client auth"
        ]
    },
    "peer": {
        "expiry": "8760h",
        "usages": [
            "signing",
            "key encipherment",
            "server auth",
            "client auth"
        ]
    }
}
```

这里为了简单只用一个 profile ，服务端，客户端，peer 公用一个证书。
**ca-cofnig.json**

```json
{
    "signing": {
        "default": {
            "expiry": "87600h"
        },
        "profiles": {
            "etcd": {
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
```


**ca-csr.json**

```json
{
    "CN": "etcd",
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "US",
            "L": "CA",
            "ST": "San Francisco",
            "O": "Etcd"
        }
    ]
}
```

### 生成 CA
```shell
cfssl gencert -initca ca-csr.json |cfssljson -bare ca-etcd -
```
执行完成后得到以下文件
```
ca-etcd-key.pem
ca-etcd.csr
ca-etcd.pem
```

- ca-etcd-key.pem 妥善保管此文件，此密钥允许在CA内创建任何类型的证书



## 3. 生成 server 端证书
```
cfssl print-defaults csr > etcd-server.json
```

服务器证书最重要的值是 **CN** 和 **hosts**，必须替换它们。


```
...
 "CN": "etcd1",
    "hosts": [
        "192.168.122.68",
        "etcd.example.com",
        "etcd1"
    ],
...
```

生成 server 证书和私钥：

```
cfssl gencert -ca-ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=etcd etcd-server.json | cfssljson -bare cert-etcd-server
```

将生成以下文件：
```
cert-etcd-server-key.pem
cert-etcd-server.csr
cert-etcd-server.pem
```



## 4. 生成对等证书

```
cfssl print-defaults csr > member1.json
```

替换 **CN** 和 **hosts** 。

```
...
    "CN": "member1",
    "hosts": [
        "192.168.122.101",
        "member1.example.com",
        "member1"
    ],
...
````

生成 member1 的证书和私钥：

``` shell
cfssl gencert -ca-ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=etcd member1.json | cfssljson -bare cert-member1
```


## 5. 生成客户端证书

```shell
cfssl print-defaults csr > client.json
```

客户端证书，可以忽略 hosts ，仅将 CN 设置为客户端值。

```
...
    "CN": "client",
...
````

生成 client 的证书和私钥：

```shell
cfssl gencert -ca-ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=etcd client.json | cfssljson -bare client
```

## 6. 验证数据

```
openssl x509 -in ca.pem -text -noout
openssl x509 -in server.pem -text -noout
openssl x509 -in client.pem -text -noout
```


## 7. 注意点
- 将 ca-key.pem 存储在安全的地方
- 确保文件安全，设置适当的文件权限。 chmod 0600 server-key.pem
- * 为通配符地址，可以工作在任何机器上，简化证书流程，但是会增加安全风险。
---
title: "ETCD 集群搭建"
linkTitle: "ETCD 集群搭建"
weight: 230
description: >
    ETCD 集群搭建
---

最主要的还是阅读官方文档，下面演示一些操作。生成证书可以参考上面 cfssl使用 文章。

下面演示在一个节点起三个etcd节点。


## 1.没有 TLS

```
etcd --name etcd1 \
  --data-dir /root/etcd/etcd1 \
  --initial-advertise-peer-urls http://172.31.239.82:12380 \
  --listen-peer-urls http://172.31.239.82:12380 \
  --listen-client-urls http://172.31.239.82:12379,http://127.0.0.1:12379 \
  --advertise-client-urls http://172.31.239.82:12379 \
  --initial-cluster-token token \
  --initial-cluster etcd1=http://172.31.239.82:12380,etcd2=http://172.31.239.82:22380,etcd3=http://172.31.239.82:32380 \
  --initial-cluster-state new


etcd --name etcd2 \
  --data-dir /root/etcd/etcd2 \
  --initial-advertise-peer-urls http://172.31.239.82:22380 \
  --listen-peer-urls http://172.31.239.82:22380 \
  --listen-client-urls http://172.31.239.82:22379,http://127.0.0.1:22379 \
  --advertise-client-urls http://172.31.239.82:22379 \
  --initial-cluster-token token \
  --initial-cluster etcd1=http://172.31.239.82:12380,etcd2=http://172.31.239.82:22380,etcd3=http://172.31.239.82:32380 \
  --initial-cluster-state new


etcd --name etcd3 \
    --data-dir /root/etcd/etcd3 \
  --initial-advertise-peer-urls http://172.31.239.82:32380 \
  --listen-peer-urls http://172.31.239.82:32380 \
  --listen-client-urls http://172.31.239.82:32379,http://127.0.0.1:32379 \
  --advertise-client-urls http://172.31.239.82:32379 \
  --initial-cluster-token token \
  --initial-cluster etcd1=http://172.31.239.82:12380,etcd2=http://172.31.239.82:22380,etcd3=http://172.31.239.82:32380 \
  --initial-cluster-state new
```


```
cd etcd/ssl
etcdctl  --endpoints=http://172.31.239.82:12379 member list -w table
etcdctl  --endpoints=http://172.31.239.82:12379 endpoint status -w table --cluster

```

## 2. 有 TLS
```
rm -fr  /root/etcd/etcd1/*
etcd --name etcd1 \
  --data-dir /root/etcd/etcd1 \
  --initial-advertise-peer-urls https://172.31.239.82:12380 \
  --listen-peer-urls https://172.31.239.82:12380 \
  --listen-client-urls https://172.31.239.82:12379,https://127.0.0.1:12379 \
  --advertise-client-urls https://172.31.239.82:12379 \
  --initial-cluster-token token \
  --initial-cluster etcd1=https://172.31.239.82:12380,etcd2=https://172.31.239.82:22380,etcd3=https://172.31.239.82:32380 \
  --initial-cluster-state new \
  --client-cert-auth \
  --peer-client-cert-auth \
  --trusted-ca-file "/root/etcd/ssl/ca-etcd.pem" \
  --cert-file "/root/etcd/ssl/cert-etcd.pem" \
  --key-file "/root/etcd/ssl/cert-etcd-key.pem" \
  --peer-trusted-ca-file "/root/etcd/ssl/ca-etcd.pem" \
  --peer-cert-file "/root/etcd/ssl/cert-etcd.pem" \
  --peer-key-file "/root/etcd/ssl/cert-etcd-key.pem" \
  --cipher-suites "TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_AES_256_GCM_SHA384"

rm -fr  /root/etcd/etcd2/*
etcd --name etcd2 \
  --data-dir /root/etcd/etcd2 \
  --initial-advertise-peer-urls https://172.31.239.82:22380 \
  --listen-peer-urls https://172.31.239.82:22380 \
  --listen-client-urls https://172.31.239.82:22379,https://127.0.0.1:22379 \
  --advertise-client-urls https://172.31.239.82:22379 \
  --initial-cluster-token token \
  --initial-cluster etcd1=https://172.31.239.82:12380,etcd2=https://172.31.239.82:22380,etcd3=https://172.31.239.82:32380 \
  --initial-cluster-state new \
  --client-cert-auth \
  --peer-client-cert-auth \
  --trusted-ca-file "/root/etcd/ssl/ca-etcd.pem" \
  --cert-file "/root/etcd/ssl/cert-etcd.pem" \
  --key-file "/root/etcd/ssl/cert-etcd-key.pem" \
  --peer-trusted-ca-file "/root/etcd/ssl/ca-etcd.pem" \
  --peer-cert-file "/root/etcd/ssl/cert-etcd.pem" \
  --peer-key-file "/root/etcd/ssl/cert-etcd-key.pem" \
  --cipher-suites "TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_AES_256_GCM_SHA384"

rm -fr  /root/etcd/etcd3/*
etcd --name etcd3 \
    --data-dir /root/etcd/etcd3 \
  --initial-advertise-peer-urls https://172.31.239.82:32380 \
  --listen-peer-urls https://172.31.239.82:32380 \
  --listen-client-urls https://172.31.239.82:32379,https://127.0.0.1:32379 \
  --advertise-client-urls https://172.31.239.82:32379 \
  --initial-cluster-token token \
  --initial-cluster etcd1=https://172.31.239.82:12380,etcd2=https://172.31.239.82:22380,etcd3=https://172.31.239.82:32380 \
  --initial-cluster-state new \
  --client-cert-auth \
  --peer-client-cert-auth \
  --trusted-ca-file "/root/etcd/ssl/ca-etcd.pem" \
  --cert-file "/root/etcd/ssl/cert-etcd.pem" \
  --key-file "/root/etcd/ssl/cert-etcd-key.pem" \
  --peer-trusted-ca-file "/root/etcd/ssl/ca-etcd.pem" \
  --peer-cert-file "/root/etcd/ssl/cert-etcd.pem" \
  --peer-key-file "/root/etcd/ssl/cert-etcd-key.pem" \
  --cipher-suites "TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_AES_256_GCM_SHA384"
```


```
cd etcd/ssl
etcdctl  --endpoints=https://172.31.239.82:12379  --cacert=ca-etcd.pem --cert=cert-etcd.pem  --key=cert-etcd-key.pem member list -w table
```
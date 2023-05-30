---
title: "服务器配置 SSH 公钥登陆"
linkTitle: "服务器配置 SSH 公钥登陆"
weight: 100
description: >
  服务器配置 SSH 公钥登陆
---

{{% pageinfo %}}
 服务器配置 SSH 公钥登陆
{{% /pageinfo %}}



1. Xshell 工具可以生成公私钥对，输入密码加密。
2. 在服务器端编辑 **/etc/ssh/sshd_conf** 。
    ```
    RSAAuthentication yes #开启RSA认证
    PubkeyAuthentication yes #开启公钥认证
    AuthorizedKeysFile .ssh/authorized_keys #设置公钥的保存位置，默认为用户目录下的.ssh目录，没有可自行创建。
    ```
3. 将保存好的 **.pub 公钥文件** 上传到服务器，保存在 **~/.ssh/authorized_keys** 文件中。
4. 设置权限
    ```
    chmod 700 .ssh
    chmod 600 ~/.ssh/authorized_keys
    ```
5. 重启 sshd 服务以使配置生效
    ```
    service sshd restart
    ```
6. 在测试公钥登陆成功后可以再次编辑 **/etc/ssh/sshd_config** 文件，关闭密码登陆，提高安全性。
    ```
    PasswordAuthentication no
    ```
---
title: "minio"
linkTitle: "miio"
weight: 226
description: >
    企业级开源的对象存储。
---

[服务端和客户端cli工具下载地址](https://min.io/download#/kubernetes)


## 客户端操作
一般使用客户端命令进行备份操作。


```shell
# 添加远程，需要 access key
mc config host add my-minio https://xxx --api S3v4

# 上传
mc cp {local/path} my-minio/backup/  #可以用 * 上传目录下所有的文件

# 下载所有backup backut下的文件到本地当前目录
mc cp --recursive  my-minio/backup .

# 查看所有的bucket
mc ls my-minio

```


```
NAME:
  mc - MinIO Client for cloud storage and filesystems.

USAGE:
  mc [FLAGS] COMMAND [COMMAND FLAGS | -h] [ARGUMENTS...]

COMMANDS:
  alias      set, remove and list aliases in configuration file
  ls         list buckets and objects
  mb         make a bucket
  rb         remove a bucket
  cp         copy objects
  mirror     synchronize object(s) to a remote site
  cat        display object contents
  head       display first 'n' lines of an object
  pipe       stream STDIN to an object
  share      generate URL for temporary access to an object
  find       search for objects
  sql        run sql queries on objects
  stat       show object metadata
  mv         move objects
  tree       list buckets and objects in a tree format
  du         summarize disk usage recursively
  retention  set retention for object(s)
  legalhold  manage legal hold for object(s)
  diff       list differences in object name, size, and date between two buckets
  rm         remove objects
  version    manage bucket versioning
  ilm        manage bucket lifecycle
  encrypt    manage bucket encryption config
  event      manage object notifications
  watch      listen for object notification events
  undo       undo PUT/DELETE operations
  policy     manage anonymous access to buckets and objects
  tag        manage tags for bucket and object(s)
  replicate  configure server side bucket replication
  admin      manage MinIO servers
  update     update mc to latest release

GLOBAL FLAGS:
  --autocompletion              install auto-completion for your shell
  --config-dir value, -C value  path to configuration folder (default: "/root/.mc")
  --quiet, -q                   disable progress bar display
  --no-color                    disable color theme
  --json                        enable JSON lines formatted output
  --debug                       enable debug output
  --insecure                    disable SSL certificate verification
  --help, -h                    show help
  --version, -v                 print the version

TIP:
  Use 'mc --autocompletion' to enable shell autocompletion

VERSION:
  RELEASE.2021-03-23T05-46-11Z
```
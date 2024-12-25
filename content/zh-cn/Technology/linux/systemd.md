---
title: "Systemd"
linkTitle: "Systemd"
weight: 98
description: >
  Systemd 是一系列工具的集合，其作用也远远不仅四启动操作系统，还接管了后台服务、结束、状态查询，以及日志归档、设备管理、段元管理、定时任务等许多职责，并支持通过特定事件和特定端口数据触发的任务。
---

{{% pageinfo %}}
Systemd 是一系列工具的集合，包括了一个专用的系统日志管理服务：**Journald**。这个服务的初衷是克服 **syslog** 服务的日志内容容易伪造和日志格式不统一等缺点。
**Journald** 使用二进制格式报错所有的日志信息，因此日志信息很难被伪造。提供 **journalctl** 命令来查看日志信息。
{{% /pageinfo %}}


## Unit 文件
Systemd 可以管理所有的系统资源，不同的资源统称为 Unit（单位）。

Unit文件统一了过去不同系统资源配置格式，例如服务的启/停、定时任务、设备自动挂载、网络配置、虚拟内存配置等。而 Systemd 通过不同的文件后缀区分这些配置文件。

### 支持的12种 Unit 文件类型
- .timer：配置在特定时间触发的任务，替代了 Crontab的功能
- .service：封装守护进程的启动、停止、重启和重载操作，是最常见的一种 Unit 文件
- .target：用于对 Unit 文件进行逻辑分组，引导其他 Unit 的执行

- .automount：用于控制自动挂载文件系统，相当于autofs 服务。
- .device： 对于/dev目录下的设备，主要用于定义设备之间的依赖关系
- .mount：定义系统结构层次中的一个挂载点，替代过去的 /etc/fstab 配置文件
- .scope： Systemd运行时产生的文件，描述一些系统服务的分组信息
- .slice：用于表示一个 cGroup 的树，通常用户不会自己创建
- .snapshot
- .socket
- .swap

### 目录
按照约定，放置在指定的是哪个系统目录之一中。优先级高的目录里的文件会被使用。
- /etc/systemd/system ：系统或者用户自定义的配置文件（优先级最高）
- /run/systemd/system ：软件运行时生成的配置文件
- /usr/lib/systemd/system ：系统或者第三方软件安装时添加的配置文件

### 文件结构
例如：
```
[Unit]
Description=Hello World
After=docker.service
Requires=docker.service
[Service]
TimeoutStartSec=0
ExecStartPre=-/usr/bin/docker kill busybox1
ExecStartPre=-/usr/bin/docker rm busybox1
ExecStartPre=/usr/bin/docker pull busybox
ExecStart=/usr/bin/docker run --name busybox1 busybox /bin/ sh -c "while true; do echo Hello World; sleep 1; done"
ExecStop="/usr/bin/docker stop busybox1"
ExecStopPost="/usr/bin/docker rm busybox1"
[Install]
WantedBy=multi-user.target
```

Unit 文件可以分为三个配置区段：
- Unit 段：所有 Unit 文件通用，配置服务的描述、依赖和随系统启动的方式。
- Service 段：后缀为 **.service** 特有的，用于定义服务的具体管理和操作方法。
- Install 段: 和 Unit 段 作用相同。

### Unit 段
- Description
- Documentation
- Requires
- Wants
- After
- Before
- Binds To
- Part Of
- OnFailure
- Conflicts

### Install 段
这部分配置的目标通常是特定运行目标的 **.target** 文件，用来使得服务在系统启动时自动运行。这个区段可以包含三种启动约束：
- WantedBy
- RequiredBy
- Also 当前 Unit enable/disabled时，同时 enable/disable 其他 Unit
- Alias 启动的别名

> systemctl list-units --type=target  # 获取当前正在使用的运行目标

### Service 段

声明周期相关：
- type ：启动时的进程行为
    - simple： 默认值，执行 execstart 指定的命令，启动主进程
    - forking 从父进程创建子进程，创建后父进程会立即突出
    - oneshot 一次性进程， systemd 会等当前服务退出，再继续往下执行
    - dbus 
    - notify 当前服务启完毕，通知 systemd，再继续往下执行
    - idle  若有其他任务执行完毕，当前服务才会运行
- RemainAfterExit
- ExecStart
- ExecStartPre
- ExecStartPos
- ExecReload
- ExecStop
- ExecStopPost
- RestartSec
- Restart 重新启动， always on-success on-failure on-abnormal on-abort on-watchdog
- TimeoutStartSec  启动服务时等待的秒数，一般docker容器拉镜像时间较长设置0，关闭超时检测
- TimeoutStopSec  停止服务时的等待秒数，如果超过这个时间仍然没有停止，syystemd会使用 sigkill 信号强行杀死服务的进程。


服务上下文配置相关
- Environment 为服务指定环境变量
- EnvironmentFile 
- Nice 服务进程的优先级，越小越高，默认为0，范围是 -20~19
- WorkingDirectory
- RootDirectory 服务进程的根目录
- User
- Group

> ：如果在 ExecStart、ExecStop 等属性中使用了 Linux 命令，则必须要写出完整的绝对路径。对于 ExecStartPre 和 ExecStartPost 辅助命令，若前面有个 “-” 符号，表示忽略这些命令的出错。




## Systemd 资源管理

Systemctl 命令：
```
# 列出正在运行的 Unit
systemctl list-units

# 列出所有Unit，包括没有找到配置文件的或者启动失败的
systemctl list-units --all

# 列出所有没有运行的 Unit
systemctl list-units --all --state=inactive

# 列出所有加载失败的 Unit
systemctl list-units --failed

# 列出所有正在运行的、类型为 service 的 Unit
systemctl list-units --type=service

# 查看 Unit 配置文件的内容
systemctl cat docker.service
```


查看 Unit 的状态：
```
# 显示系统状态
systemctl status

# 显示单个 Unit 的状态
systemctl status bluetooth.service
```

```
# 立即启动一个服务
sudo systemctl start apache.service

# 立即停止一个服务
sudo systemctl stop apache.service

# 重启一个服务
sudo systemctl restart apache.service

# 杀死一个服务的所有子进程
sudo systemctl kill apache.service

# 重新加载一个服务的配置文件
sudo systemctl reload apache.service

# 重载所有修改过的配置文件
sudo systemctl daemon-reload

# 显示某个 Unit 的所有底层参数
systemctl show httpd.service

# 显示某个 Unit 的指定属性的值
systemctl show -p CPUShares httpd.service

# 设置某个 Unit 的指定属性
sudo systemctl set-property httpd.service CPUShares=500
```

查看 Unit 的依赖关系：

```
# 列出所有依赖，默认不会列出 target 类型
systemctl list-dependencies nginx.service
# 列出所有依赖，包括 target 类型
systemctl list-dependencies --all nginx.service
```

## Systemd 工具集

- systemctl：用于检查和控制各种系统服务和资源的状态
- bootctl：用于查看和管理系统启动分区
- hostnamectl：用于查看和修改系统的主机名和主机信息
- journalctl：用于查看系统日志和各类应用服务日志
- localectl：用于查看和管理系统的地区信息
- loginctl：用于管理系统已登录用户和 Session 的信息
- machinectl：用于操作 Systemd 容器
- timedatectl：用于查看和管理系统的时间和时区信息
- systemd-analyze 显示此次系统启动时运行每个服务所消耗的时间，可以用于分析系统启动过程中的性能瓶颈
- systemd-ask-password：辅助性工具，用星号屏蔽用户的任意输入，然后返回实际输入的内容
- systemd-cat：用于将其他命令的输出重定向到系统日志
- systemd-cgls：递归地显示指定 CGroup 的继承链
- systemd-cgtop：显示系统当前最耗资源的 CGroup 单元
- systemd-escape：辅助性工具，用于去除指定字符串中不能作为 Unit 文件名的字符
- systemd-hwdb：Systemd 的内部工具，用于更新硬件数据库
- systemd-delta：对比当前系统配置与默认系统配置的差异
- systemd-detect-virt：显示主机的虚拟化类型
- systemd-inhibit：用于强制延迟或禁止系统的关闭、睡眠和待机事件
- systemd-machine-id-setup：Systemd 的内部工具，用于给 Systemd 容器生成 ID
- systemd-notify：Systemd 的内部工具，用于通知服务的状态变化
- systemd-nspawn：用于创建 Systemd 容器
- systemd-path：Systemd 的内部工具，用于显示系统上下文中的各种路径配置
- systemd-run：用于将任意指定的命令包装成一个临时的后台服务运行
- systemd-stdio- bridge：Systemd 的内部 工具，用于将程序的标准输入输出重定向到系统总线
- systemd-tmpfiles：Systemd 的内部工具，用于创建和管理临时文件目录
- systemd-tty-ask-password-agent：用于响应后台服务进程发出的输入密码请求
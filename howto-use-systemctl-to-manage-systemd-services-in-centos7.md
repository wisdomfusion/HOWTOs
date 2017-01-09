# How To Use Systemctl to Manage Systemd Services in CentOS 7

## 0. 简介

CentOS 7 使用 systemd 替换了 SysV。Systemd 目的是要取代 Unix 时代以来一直在使用的 init 系统，兼容 SysV 和 LSB 的启动脚本，而且够在进程启动过程中更有效地引导加载服务。

systemd 特性：

- 支持并行化任务
- 同时采用 socket 式与 D-Bus 总线式激活服务；
- 按需启动守护进程（daemon）；
- 利用 Linux 的 cgroups 监视进程；
- 支持快照和系统恢复；
- 维护挂载点和自动挂载点；
- 各服务间基于依赖关系进行精密控制。

检视和控制systemd的主要命令是 `systemctl`。该命令可用于查看系统状态和管理系统及服务。详见 `man systemctl`。在 `systemctl` 参数中添加 `-H <用户名>@<主机名>` 可以实现对其他机器的远程控制。该过程使用 SSH 连接。

## 1. 分析系统状态

### 1.1. 查看系统状态

打印当前单元（unit）列表：

    systemctl list-units

关于单元（unit）的详细说明见下文。

等效于：

    systemctl

查看所有单元：

    systemctl list-units --all

查看所有的处于未激活状态的单元：

    systemctl list-units --all --state=inactive

查看运行失败的单元：

    systemctl --failed

查看服务单元：

    systemctl list-units --type=service

### 1.2. 检查单元状态

使用 `status` 子命令查看服务的状态：

    systemctl status <unit>

使用 `is-active` 查看服务是否是运行（running）的状态：

    systemctl is-active <unit>

使用 `is-enabled` 检查服务是否被启用：

    systemctl is-enabled <unit>

还可以检查服务是否处于失败状态：

    systemctl is-failed <unit>

### 1.3. 列出所有单元文件

所有可用的单元文件存放在 `/usr/lib/systemd/system` 和 `/etc/systemd/system` 目录（**后者优先级更高**）。

- `/usr/lib/systemd/system` 软件包安装的单元
- `/etc/systemd/system` 系统管理员安装的单元

查看所有已安装服务：

    systemctl list-unit-files

## 2. 单元（unit）管理

### 2.1. 单元说明

一个单元配置文件可以描述如下内容之一：系统服务（`.service`）、挂载点（`.mount`）、sockets（`.sockets`） 、系统设备（`.device`）、交换分区（`.swap`）、文件路径（`.path`）、启动目标（`.target`）、由 systemd 管理的计时器（`.timer`）。详情参阅 `man systemd.unit`。

使用 `systemctl` 控制单元时，通常需要使用单元文件的全名，包括扩展名（例如 `sshd.service`）。但是有些单元可以在 `systemctl` 中使用简写方式。

- 如果无扩展名，`systemctl` 默认把扩展名当作 `.service`。例如 `nginx` 和 `nginx.service` 是等价的。
- 挂载点会自动转化为相应的 `.mount` 单元。例如 `/home` 等价于 `home.mount`。
- 设备会自动转化为相应的 `.device` 单元，所以 `/dev/sda2` 等价于 `dev-sda2.device`。

### 2.2. 单元的基本管理

立即启动单元：

    systemctl start <unit>

立即停止单元：

    systemctl stop <unit>

重启单元：

    systemctl restart <unit>

重新加载配置：

    systemctl reload <unit>

输出单元运行状态：

    systemctl status <unit>

检查单元是否配置为自动启动：

    systemctl is-enabled <unit>

开机自动激活单元：

    systemctl enable <unit>

取消开机自动激活单元：

    systemctl disable <unit>

禁用一个单元（禁用后，间接启动也是不可能的）：

    systemctl mask <unit>

取消禁用一个单元：

    systemctl unmask <unit>

显示单元的手册页（必须由单元文件提供）：

    systemctl help <unit>

重新载入 systemd，扫描新的或有变动的单元：

    systemctl daemon-reload


### 2.3. 查看单元文件

```sh
systemctl cat sshd.service
```

示例输出：

    # /usr/lib/systemd/system/sshd.service
    [Unit]
    Description=OpenSSH server daemon
    Documentation=man:sshd(8) man:sshd_config(5)
    After=network.target sshd-keygen.service
    Wants=sshd-keygen.service
    
    [Service]
    EnvironmentFile=/etc/sysconfig/sshd
    ExecStart=/usr/sbin/sshd -D $OPTIONS
    ExecReload=/bin/kill -HUP $MAINPID
    KillMode=process
    Restart=on-failure
    RestartSec=42s
    
    [Install]
    WantedBy=multi-user.target

### 2.4. 查看单元依赖

命令：

    systemctl list-dependencies sshd.service

示例输出：

```
sshd.service
├─system.slice
└─basic.target
  ├─microcode.service
  ├─rhel-autorelabel-mark.service
  ├─rhel-autorelabel.service
  ├─rhel-configure.service
  ├─rhel-dmesg.service
  ├─rhel-loadmodules.service
  ├─paths.target
  ├─slices.target
...
```

### 2.5. 检查单元属性

命令：

    systemctl show sshd.service

示例输出：

```
Id=sshd.service
Names=sshd.service
Requires=basic.target
Wants=system.slice
WantedBy=multi-user.target
Conflicts=shutdown.target
Before=shutdown.target multi-user.target
After=syslog.target network.target auditd.service systemd-journald.socket basic.target system.slice
Description=OpenSSH server daemon
...
```

### 2.6. 修改已有单元文件

    systemctl edit --full sshd.service

### 2.7. 创建新单元文件

CentOS 系统系统脚本目录在 `/usr/lib/systemd` 或 `/lib/systemd`中（`/lib` 只是 `/usr/lib` 目录的一个链接，指向同一个目录），有系统（system）和用户（user）之分，如需要在3运行时运行的服务，存放在系统（system）服务里 `/usr/lib/systemd/system`，反之，用户登录后才能运行的程序，存放在用户（user）服务里，服务以 `.service` 结尾。

这里以 nginx 服务文件为例，建立服务文件。

```sh
cat > /usr/lib/systemd/system/nginx.service <<\EOF
[Unit]
Description=The NGINX HTTP and reverse proxy server
After=syslog.target network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/usr/local/webserver/nginx/logs/nginx.pid
ExecStartPre=/usr/local/webserver/nginx/sbin/nginx -t
ExecStart=/usr/local/webserver/nginx/sbin/nginx
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target
EOF
```

参考 [NGINX systemd service file][nginx.service]，其中，nginx pid 路径和 nginx 可执行程序的路径按实际情况修改之，上面是按照我编译安装后对应的路径改的 service 文件。更多其他服务的 service 文件请看我的这个 Git Repo :point_right: [systemd-services](https://github.com/WisdomFusion/systemd-services)。

[nginx.service]: https://www.nginx.com/resources/wiki/start/topics/examples/systemd/

service 文件字段说明：

    [Unit] 服务的说明
    Description 描述服务
    After 描述服务类别

    [Service] 服务运行参数的设置
    Type=forking 是后台运行的形式
    ExecStart 为服务的具体运行命令
    ExecReload 为重启命令
    ExecStop 为停止命令
    PrivateTmp=True 表示给服务分配独立的临时空间

    注意：[Service]的启动、重启、停止命令全部要求使用绝对路径

    [Install] 服务安装的相关设置，可设置为多用户

详细说明请参看 `man systemd.service`，或参看 [systemd.service 中文手册](http://www.jinbuguo.com/systemd/systemd.service.html) 下面针对服务类型作说明：

**服务类型**

编写自定义的 service 文件时，可以选择几种不同的服务启动方式。启动方式可通过配置文件 `[Service]` 段中的 `Type=` 参数进行设置。

- **Type=simple**（默认值）：systemd 认为该服务将立即启动。服务进程不会 fork。如果该服务要启动其他服务，不要使用此类型启动，除非该服务是 socket 激活型。
- **Type=forking**：systemd 认为当该服务进程 fork，且父进程退出后服务启动成功。对于常规的守护进程（daemon），除非你确定此启动方式无法满足需求，使用此类型启动即可。使用此启动类型应同时指定 PIDFile=，以便 systemd 能够跟踪服务的主进程。
- **Type=oneshot**：这一选项适用于只执行一项任务、随后立即退出的服务。可能需要同时设置 RemainAfterExit=yes 使得 systemd 在服务进程退出之后仍然认为服务处于激活状态。
- **Type=notify**：与 Type=simple 相同，但约定服务会在就绪后向 systemd 发送一个信号。这一通知的实现由 libsystemd-daemon.so 提供。
- **Type=dbus**：若以此方式启动，当指定的 BusName 出现在 DBus 系统总线上时，systemd 认为服务就绪。
- **Type=idle**: systemd 会等待所有任务(Jobs)处理完成后，才开始执行 idle 类型的单元。除此之外，其他行为和 **Type=simple** 类似。


## 3. 使用目标（target）管理系统启动级别（runlevel）

启动级别（runlevel）是一个旧的概念。现在，systemd 引入了一个和启动级别功能相似又不同的概念——目标（target）。不像数字表示的启动级别，每个目标都有名字和独特的功能，并且能同时启用多个。一些目标继承其他目标的服务，并启
动新服务。systemd 提供了一些模仿 sysvinit 启动级别的目标，仍可以使用旧的 `telinit <启动级别>` 命令切换。

### 3.1. 获取当前目标

    systemctl list-units --type=target

### 3.2. 创建新目标

启动级别 0、1、3、5、6 都被赋予特定用途，并且都对应一个 systemd 的目标。然而，没有什么很好的移植用户定义的启动级别（2、4）的方法。要实现类似功能，可以以原有的启动级别为基础，创建一个新的目标 `/etc/systemd/system/<新目标>`（可以参考 `/usr/lib/systemd/system/graphical.target`），创建 `/etc/systemd/system/<新目标>.wants` 目录，向其中加入额外服务的链接（指向 `/usr/lib/systemd/system` 中的单元文件）。

目标表

| SysV 启动级别 | Systemd 目标                                          | 注释                                                    |
|---------------|-------------------------------------------------------|---------------------------------------------------------|
|             0 | runlevel0.target, poweroff.target                     | 中断系统（halt）                                        |
|  1, s, single | runlevel1.target, rescue.target                       | 单用户模式                                              |
|          2, 4 | runlevel2.target, runlevel4.target, multi-user.target | 用户自定义启动级别，通常识别为级别3。                   |
|             3 | runlevel3.target, multi-user.target                   | 多用户，无图形界面。用户可以通过终端或网络登录。        |
|             5 | runlevel5.target, graphical.target                    | 多用户，图形界面。继承级别3的服务，并启动图形界面服务。 |
|             6 | runlevel6.target, reboot.target                       | 重启                                                    |
|     emergency | emergency.target                                      | 急救模式（Emergency shell）                             |

### 3.3. 切换启动级别/目标

systemd 中，启动级别通过“目标单元”访问。通过如下命令切换：

    systemctl isolate graphical.target

该命令对下次启动无影响。等价于 `telinit 3` 或 `telinit 5`。

### 3.4. 修改默认启动级别/目标

开机启动进的目标是 `default.target`，默认链接到 `graphical.target`（相当于原来的runlevel 5）。可以通过内核参数更改默认启动级别：

- systemd.unit=multi-user.target （相当于 level 3）
- systemd.unit=rescue.target （相当于 level 1）

另一个方法是修改 default.target。可以通过 systemctl 修改它：

    systemctl set-default multi-user.target

要覆盖已经设置的default.target，请使用 force:

    systemctl set-default -f multi-user.target

可以在 `systemctl` 的输出中看到命令执行的效果：链接 `/etc/systemd/system/default.target` 被创建，指向新的默认 runlevel。

### 4. 电源管理

重启：

    systemctl reboot

退出系统并停止电源：

    systemctl poweroff

待机：

    systemctl suspend

休眠：

    systemctl hibernate

混合休眠模式（同时休眠到硬盘并待机）：

    systemctl hybrid-sleep

## 5. 总结

怎么样？systemd 有够强大吧，除此之外，systemd 还有计时器和日志功能等功能：

- 定时器是以 .timer 为后缀的配置文件，记录由system的里面由时间触发的动作, 定时器可以替代 cron 的大部分功能。
- systemd 提供了自己日志系统（logging system），称为 journal. 使用 systemd 日志，无需额外安装日志服务（syslog）。

限于篇幅，本文不在展开，详细用法说明请参看 `man systemd.timer` 和 `man journalctl`。

有些同学可能觉得 `systemctl` 的命令加上子命令和参数，太:cry:长:weary:了，安上 bash-completion 可以加快命令的输入：

    yum -y install bash-completion


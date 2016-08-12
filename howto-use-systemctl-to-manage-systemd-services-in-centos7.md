# How To Use Systemctl to Manage Systemd Services and Units

## 0. 简介

检视和控制systemd的主要命令是 `systemctl`。该命令可用于查看系统状态和管理系统及服务。详见 `man systemctl`。在 `systemctl` 参数中添加 `-H <用户名>@<主机名>` 可以实现对其他机器的远程控制。该过程使用 SSH 连接。

## 1. 服务管理

### 1.1. 启动和停止服务

启动一个服务，很简单：

    sudo systemctl start <application>.service

`systemd` 会查找 `*.service` 文件中的管理命令，所以可以简写成如下的形式：

    sudo systemctl start <application>

停止一个服务：

    sudo systemctl stop <application>.service

### 1.2. 重启服务或重新加载配置

重启一个服务：

    sudo systemctl restart <application>.service

重新加载配置而不重启服务：

    sudo systemctl reload <application>.service

如果不确定该服务有没有 `reload` 功能用于重新加载配置，那么，`systemd` 为我们准备了一个 `reload-or-restart` 命令：

    sudo systemctl reload-or-restart <application>.service

### 1.3. 启用或禁用服务

如果想让一个服务随机启动，需要用到 `enable` 命令：

    sudo systemctl enable <application>.service

该命令会

    sudo systemctl disable <application>.service

需要注意的是，启用（`enable`）服务并不意味着该服务被启动（`start`）了，需要手动启动一下。

### 1.4. 检查服务状态

使用 `status` 子命令查看服务的状态：

    systemctl status <application>.service

使用 `is-active` 查看服务是否是运行（running）的状态：

    systemctl is-active <application>.service

使用 `is-enabled` 检查服务是否被启用：

    systemctl is-enabled <application>.service

还可以检查服务是否处于失败状态：

    systemctl is-failed <application>.service

## 2. 分析系统状态

### 2.1. 打印当前 Units 列表：

打印前当前激活的单元（unit）：

    systemctl list-units

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

### 2.2. 列出所有单元文件

所有可用的单元文件存放在 `/usr/lib/systemd/system` 和 `/etc/systemd/system` 目录（**后者优先级更高**）。查看所有已安装服务：

    systemctl list-unit-files

## 3. 单元（unit）管理

### 3.1. 单元说明

一个单元配置文件可以描述如下内容之一：系统服务（`.service`）、挂载点（`.mount`）、sockets（`.sockets`） 、系统设备（`.device`）、交换分区（`.swap`）、文件路径（`.path`）、启动目标（`.target`）、由 systemd 管理的计时器（`.timer`）。详情参阅 `man systemd.unit`。

使用 `systemctl` 控制单元时，通常需要使用单元文件的全名，包括扩展名（例如 `sshd.service`）。但是有些单元可以在 `systemctl` 中使用简写方式。

- 如果无扩展名，`systemctl` 默认把扩展名当作 `.service`。例如 `nginx` 和 `nginx.service` 是等价的。
- 挂载点会自动转化为相应的 `.mount` 单元。例如 `/home` 等价于 `home.mount`。
- 设备会自动转化为相应的 `.device` 单元，所以 `/dev/sda2` 等价于 `dev-sda2.device`。

### 3.2. 单元和基本管理

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


### 3.3. 查看单元文件

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

### 3.4. 查看单元依赖

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

### 3.5. 检查单元属性

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

### 3.6. 修改已有单元文件

    systemctl edit --full sshd.service

### 3.7. 创建新单元文件

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

参考 [NGINX systemd service file][nginx.service]，其中，nginx pid 路径和 nginx 可执行程序的路径按实际情况修改之，上面是按照我编译安装后对应的路径改的 service 文件。

    [Unit]:服务的说明
    Description:描述服务
    After:描述服务类别
    [Service]服务运行参数的设置
    Type=forking是后台运行的形式
    ExecStart为服务的具体运行命令
    ExecReload为重启命令
    ExecStop为停止命令
    PrivateTmp=True表示给服务分配独立的临时空间
    注意：[Service]的启动、重启、停止命令全部要求使用绝对路径
    [Install]服务安装的相关设置，可设置为多用户

更多其他服务的 service 文件请看我的这个 Git Repo：[systemd-services](https://github.com/eventhorizonpl/systemd-services)

[nginx.service]: https://www.nginx.com/resources/wiki/start/topics/examples/systemd/

## 4. 使用目标（target）管理系统启动级别（runlevel）

启动级别（runlevel）是一个旧的概念。现在，systemd 引入了一个和启动级别功能相似又不同的概念——目标（target）。不像数字表示的启动级别，每个目标都有名字和独特的功能，并且能同时启用多个。一些目标继承其他目标的服务，并启
动新服务。systemd 提供了一些模仿 sysvinit 启动级别的目标，仍可以使用旧的 `telinit <启动级别>` 命令切换。

### 4.1. 获取当前目标

    systemctl list-units --type=target

### 4.2. 创建新目标

启动级别 0、1、3、5、6 都被赋予特定用途，并且都对应一个 systemd 的目标。然而，没有什么很好的移植用户定义的启动级别（2、4）的方法。要实现类似功能，可以以原有的启动级别为基础，创建一个新的目标 /etc/systemd/system/<新目标>（可以参考 /usr/lib/systemd/system/graphical.target），创建 /etc/systemd/system/<新目标>.wants 目录，向其中加入额外服务的链接（指向 /usr/lib/systemd/system/ 中的单元文件）。

目标表

|SysV 启动级别|Systemd 目标|注释|
|------------|------------|----|
|0|runlevel0.target, poweroff.target|中断系统（halt）|
|1, s, single|runlevel1.target, rescue.target|单用户模式|
|2, 4|runlevel2.target, runlevel4.target, multi-user.target|用户自定义启动级别，通常识别为级别3。|
|3|runlevel3.target, multi-user.target|多用户，无图形界面。用户可以通过终端或网络登录。|
|5|runlevel5.target, graphical.target|多用户，图形界面。继承级别3的服务，并启动图形界面服务。|
|6|runlevel6.target, reboot.target|重启|
|emergency|emergency.target|急救模式（Emergency shell）|

### 4.3. 切换启动级别/目标

systemd 中，启动级别通过“目标单元”访问。通过如下命令切换：

    systemctl isolate graphical.target

该命令对下次启动无影响。等价于 `telinit 3` 或 `telinit 5`。

### 4.4. 修改默认启动级别/目标

开机启动进的目标是 `default.target`，默认链接到 `graphical.target`（相当于原来的runlevel 5）。可以通过内核参数更改默认启动级别：

- systemd.unit=multi-user.target （相当于 level 3）
- systemd.unit=rescue.target （相当于 level 1）

另一个方法是修改 default.target。可以通过 systemctl 修改它：

    systemctl set-default multi-user.target

要覆盖已经设置的default.target，请使用 force:

    systemctl set-default -f multi-user.target

可以在 `systemctl` 的输出中看到命令执行的效果：链接 `/etc/systemd/system/default.target` 被创建，指向新的默认 runlevel。

### 4.4. Using Shortcuts for Important Events

    sudo systemctl rescue

    sudo systemctl halt

    sudo systemctl poweroff

    sudo systemctl reboot

## 5. 



## 6. 



## 7. 总结

conclusion











# How To Use Systemctl to Manage Systemd Services and Units

## 0. 简介

Systemd

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

## 2. 系统状态概览

### 2.1. 打印当前 Units 列表：

    systemctl list-units

    systemctl list-units --all

    systemctl list-units --all --state=inactive

    systemctl list-units --type=service

### 2.2. 列出所有 Unit 相关的文件

    systemctl list-unit-files


## 3. Unit 管理

### 3.1. 查看 Unit 文件

    systemctl cat atd.service

```
[Unit]
Description=ATD daemon

[Service]
Type=forking
ExecStart=/usr/bin/atd

[Install]
WantedBy=multi-user.target
```

### 3.2. 查看依赖

    systemctl list-dependencies sshd.service

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

. . .
```

### 3.3. 检查 Unit 属性

    systemctl show sshd.service

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

. . .
```

### 3.4. Masking and Unmasking Units

    sudo systemctl mask nginx.service

### 3.5. 编辑 Unit 文件

    sudo systemctl edit nginx.service

    sudo systemctl edit --full nginx.service

## 4. Adjusting the System State (Runlevel) with Targets

### 4.1. Getting and Setting the Default Target

    systemctl get-default

    multi-user.target

### 4.2. Listing Available Targets

systemctl list-unit-files --type=target


### 4.3. Isolating Targets

    systemctl list-dependencies multi-user.target

    sudo systemctl isolate multi-user.target

### 4.4. Using Shortcuts for Important Events

    sudo systemctl rescue

    sudo systemctl halt

    sudo systemctl poweroff

    sudo systemctl reboot

## 5. 结论

conclusion











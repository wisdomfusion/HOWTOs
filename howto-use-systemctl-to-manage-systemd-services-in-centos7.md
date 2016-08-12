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

### 3.5. 创建自定义服务文件

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

MySQL 的服务文件，参考 [Managing MySQL Server with systemd][mysql.service]，service 文件如下：

```sh
cat > /usr/lib/systemd/system/mariadb.service <<\EOF
# It's not recommended to modify this file in-place, because it will be
# overwritten during package upgrades.  If you want to customize, the
# best way is to create a file "/etc/systemd/system/mariadb.service",
# containing
#	.include /lib/systemd/system/mariadb.service
#	...make your changes here...
# or create a file "/etc/systemd/system/mariadb.service.d/foo.conf",
# which doesn't need to include ".include" call and which will be parsed
# after the file mariadb.service itself is parsed.
#
# For more info about custom unit files, see systemd.unit(5) or
# http://fedoraproject.org/wiki/Systemd#How_do_I_customize_a_unit_file.2F_add_a_custom_unit_file.3F

# For example, if you want to increase mariadb's open-files-limit to 10000,
# you need to increase systemd's LimitNOFILE setting, so create a file named
# "/etc/systemd/system/mariadb.service.d/limits.conf" containing:
#	[Service]
#	LimitNOFILE=10000

# Note: /usr/lib/... is recommended in the .include line though /lib/... 
# still works.
# Don't forget to reload systemd daemon after you change unit configuration:
# root> systemctl --system daemon-reload

[Unit]
Description=MariaDB database server
After=syslog.target
After=network.target

[Service]
Type=simple
User=mysql
Group=mysql

ExecStartPre=/usr/libexec/mariadb-prepare-db-dir %n
# Note: we set --basedir to prevent probes that might trigger SELinux alarms,
# per bug #547485
ExecStart=/usr/bin/mysqld_safe --basedir=/usr
ExecStartPost=/usr/libexec/mariadb-wait-ready $MAINPID

# Give a reasonable amount of time for the server to start up/shut down
TimeoutSec=300

# Place temp files in a secure directory, not /tmp
PrivateTmp=true

[Install]
WantedBy=multi-user.target
EOF
```

php-fpm 的服务文件：

```sh
cat > /usr/lib/systemd/system/php-fpm.service <<\EOF
# It's not recommended to modify this file in-place, because it
# will be overwritten during upgrades.  If you want to customize,
# the best way is to use the "systemctl edit" command.

[Unit]
Description=The PHP FastCGI Process Manager
After=syslog.target network.target

[Service]
Type=notify
PIDFile=/var/opt/remi/php70/run/php-fpm/php-fpm.pid
EnvironmentFile=/etc/opt/remi/php70/sysconfig/php-fpm
ExecStart=/opt/remi/php70/root/usr/sbin/php-fpm --nodaemonize
ExecReload=/bin/kill -USR2 $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target
EOF
```

[nginx.service]: https://www.nginx.com/resources/wiki/start/topics/examples/systemd/
[mysqld.service]: https://dev.mysql.com/doc/refman/5.7/en/server-management-using-systemd.html

### 3.6. 编辑 Unit 文件

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











# How to Install nginx with PHP 8 and MySQL 8 (LEMP stack) on Debian 10.9

## 配置基础系统

全新安装的 Debian GNU/Linux 系统，普通用户（这里假定是 `huzhifei`）是没有 sudo 权限的，需要切换到 `root` 账号权限，把 `huzhifei` 添加到 sudoer。运行如下命令：

    $ su -
    # apt install sudo
    # usermod -aG sudo huzhifei
    # exit
    $

此时，`huzhifei` 仍无 sudo 权限，需要重新登录一下，再使用 sudo，我们先更新一下系统安装源：

    $ ssh huzhifei@192.168.111.108
    $ sudo apt update
    [sudo] password for huzhifei:
    Hit:1 http://mirrors.ustc.edu.cn/debian buster InRelease
    Hit:2 http://mirrors.ustc.edu.cn/debian buster-updates InRelease
    Hit:3 http://security.debian.org/debian-security buster/updates InRelease
    Reading package lists... Done
    Building dependency tree
    Reading state information... Done
    All packages are up to date.

### 配置网络

修改 `/etc/network/interfaces` 网络配置文件，把默认的 DHCP 改为静态 IP 地址：

    $ sudo vi /etc/network/interfaces

修改后的文件如下：

    # This file describes the network interfaces available on your system
    # and how to activate them. For more information, see interfaces(5).

    source /etc/network/interfaces.d/*

    # The loopback network interface
    auto lo
    iface lo inet loopback

    # The primary network interface
    #allow-hotplug ens33
    #iface ens33 inet dhcp

    auto ens33
    iface ens33 inet static
      address 192.168.111.108
      netmask 255.255.255.0
      gateway 192.168.111.2
      dns-nameservers 192.168.111.2

重启网络服务使以上配置生效：

    $ sudo systemctl restart networking.service

查检 IP 地址配置是否生效：

    $ ip -c addr show ens33
    2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
        link/ether 00:0c:29:d0:1c:4e brd ff:ff:ff:ff:ff:ff
        inet 192.168.111.108/24 brd 192.168.111.255 scope global ens33
        valid_lft forever preferred_lft forever
        inet 192.168.111.202/24 brd 192.168.111.255 scope global secondary dynamic ens33
        valid_lft 1332sec preferred_lft 1332sec
        inet6 fe80::20c:29ff:fed0:1c4e/64 scope link
        valid_lft forever preferred_lft forever

### 安装常用工具

    $ sudo apt install -y net-tools wget curl vim

## 服务安装配置

### SSH 服务

安装 sshd：

    $ sudo apt install openssh-server

### 安装 ufw 防火墙工具

安装 ufw：

    $ sudo apt install ufw

必要的配置：

    $ sudo ufw default deny incoming
    $ sudo ufw default allow outgoing
    $ sudo ufw allow ssh
    $ sudo ufw enable

查看当前防火墙状态：

    $ sudo ufw status
    Status: active

    To                         Action      From
    --                         ------      ----
    22/tcp                     ALLOW       Anywhere
    22/tcp (v6)                ALLOW       Anywhere (v6)

## 安装 nginx Web 服务器

安装 nginx：

    $ sudo apt update
    $ sudo apt install -y nginx

添加防火墙规则，允许 nginx 端口访问：

    $ sudo ufw allow 'Nginx HTTP'
    $ sudo ufw status
    Status: active

    To                         Action      From
    --                         ------      ----
    22/tcp                     ALLOW       Anywhere
    Nginx HTTP                 ALLOW       Anywhere
    22/tcp (v6)                ALLOW       Anywhere (v6)
    Nginx HTTP (v6)            ALLOW       Anywhere (v6)

## 安装 MySQL 8.0

Debian 10 把 MySQL 的分支版本 MariaDB 添加到默认的 APT 源中，原生 MySQL 反而没有包含在内，所以，安装之前我们需要做些额外的工作。

### 添加 MySQL 官方 APT 源

首先，我们访问 [MySQL APT Repository](https://dev.mysql.com/downloads/repo/apt/)，获取当前最新的 MySQL APT deb 包的 URL（如 `https://repo.mysql.com//mysql-apt-config_0.8.16-1_all.deb`），此 deb 包实际上是 APT 源配置工具，运行如下命令，按提示设置 MySQL 8.0 安装源。

    $ cd /tmp
    $ wget https://repo.mysql.com//mysql-apt-config_0.8.16-1_all.deb
    $ sudo apt install -y lsb-release gnupg
    $ sudo dpkg -i mysql-apt-config*.deb

配置后，`/etc/apt/sources.list.d` 下，会多出一个 `mysql.list` 的安装源配置文件，运行 `apt update` 命令更新安装源，使之生效。

    $ sudo apt update

### 安装 mysql-server

运行如下命令安装 mysql-server

    $ sudo apt install -y mysql-server

检查 MySQL 服务状态

    $ sudo systemctl status mysql.service
    [sudo] password for huzhifei:
    ● mysql.service - MySQL Community Server
    Loaded: loaded (/lib/systemd/system/mysql.service; enabled; vendor preset: enabled)
    Active: active (running) since Tue 2021-03-30 10:21:14 CST; 3h 9min ago
        Docs: man:mysqld(8)
            http://dev.mysql.com/doc/refman/en/using-systemd.html
    Main PID: 9738 (mysqld)
    Status: "Server is operational"
        Tasks: 38 (limit: 4670)
    Memory: 348.3M
    CGroup: /system.slice/mysql.service
            └─9738 /usr/sbin/mysqld

    Mar 30 10:21:14 debian systemd[1]: Starting MySQL Community Server...
    Mar 30 10:21:14 debian systemd[1]: Started MySQL Community Server.

运行如下命令，对初次安装的 MySQL 服务作必要的安全配置（如更新 `root` 账号密码、移除测试数据库、禁止 `root` 用户远程访问等）：

    $ mysql_secure_installation

## 安装 PHP 8

### 添加 PHP 8 安装源

Debian GNU/Linux 默认只有 PHP7 的安装源，类似 MySQL 8.0，也要安装一个靠谱的 APT 源，这里选用 [sury APT 源](https://packages.sury.org/)。

    $ sudo apt update
    $ sudo apt install -y lsb-release ca-certificates apt-transport-https software-properties-common
    $ echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/sury-php.list
    $ wget -qO - https://packages.sury.org/php/apt.gpg | sudo apt-key add -

### 安装 PHP 8.0

    $ sudo apt update
    $ sudo apt install -y php8.0

安装 `php8.0` 时，会连带安装 apache2 相关的软件包和服务，本文使用 nginx，所以使用以下命令禁用 apache2：

    $ sudo systemctl stop apache2.service
    $ sudo systemctl disable apache2.service

安装必要的 PHP 扩展（本文目的是要安装 Laravel PHP 架构的开发和运行环境）：

    $ sudo apt install -y php8.0-{mysql,cli,common,mcrypt,bcmath,xml,fpm,curl,mbstring,gd,zip}

安装 composer：

    $ curl -sS https://getcomposer.org/installer | sudo  php -- --install-dir=/usr/local/bin --filename=composer
    $ sudo chmod +x /usr/local/bin/composer

### 构建 Laravel 演示项目，并添加必要的站点配置

前往 `/var/www` 构建 Laravel 演示项目，该还示项目的项目假定为 `debian.wf.com`：

    $ cd /var/www
    $ sudo composer create-project laravel/laravel debian.wf.com
    $ sudo chown -R www-data:www-data /var/www/*

修改 nginx 的默认站点配置文件 `/etc/nginx/sites-available/default`：

    $ cat /etc/nginx/sites-available/default
    server {
        listen 80;
        server_name debian.wf.com;
        root /var/www/debian.wf.com/public;

        index index.php;

        location / {
            try_files $uri $uri/ /index.php?$query_string;
        }

        location ~ \.php$ {
            include snippets/fastcgi-php.conf;
            fastcgi_pass unix:/run/php/php8.0-fpm.sock;
        }
    }

重点 PHP-FPM 和 nginx 服务使配置生效：

    sudo systemctl restart php8.0-fpm.service
    sudo systemctl restart nginx.service


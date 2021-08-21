# How to Install nginx with PHP 8 and MySQL 8 (LEMP stack) Using dnf on CentOS 8

## Enable network

```sh
cat /etc/sysconfig/network-scripts/ifcfg-ens33
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
UUID=e3d337bb-5e4c-4ab1-bbbe-e74f171c5b6b
DEVICE=ens33
ONBOOT=yes
IPADDR=192.168.111.102
NETMASK=255.255.255.0
GATEWAY=192.168.111.2
DNS1=192.168.111.2

systemctl restart NetworkManager.service
```

## hostname

```sh
hostnamectl set-hostname centos.example.com
```

## timezone

```sh
timedatectl list-timezones
timedatectl set-timezone Asia/Shanghai
```

## chronyd

```sh
dnf -y install chrony
systemctl enable chronyd.service
systemctl start chronyd.service
```

## disable SELinux

```sh
setenforce 0
sed -i 's/^SELINUX=.*$/SELINUX=disabled/' /etc/selinux/config
```

## disable firewalld

```sh
systemctl disable firewalld.service
systemctl stop firewalld.service
```

BTW, make sure you config iptables before deploy production servers.

## open file limit

```sh
ulimit -n 65535
sed -i '/DefaultLimitNOFILE=/c\DefaultLimitNOFILE=65535' /etc/systemd/system.conf
```

## Vim and other tools

```sh
dnf -y install git man wget unzip zip net-tools nmap iptraf iotop sysstat bash-completion lrzsz
```

**simple Vim configs**

```sh
dnf -y install vim-enhanced
alias vi='vim'
echo "alias vi='vim'" >> ~/.bashrc
echo "set fencs=utf-8,ucs-bom,gb18030,gbk,gb2312,cp936" >> ~/.vimrc
echo "set fileformats=unix,dos" >> ~/.vimrc
```

## epel

```sh
yum -y install epel-release
yum makecache
```

## Compile from source code

If you need tools for compiling installation, install `Development Tools` group:
```sh
dnf -y group install "Development Tools"
```

## Install nginx

```sh
cat > /etc/yum.repos.d/nginx.repo <<\EOF
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true
EOF

dnf makecache
dnf info nginx
dnf -y install nginx
systemctl enable nginx.service
systemctl start nginx.service
```

## Install epel and remi repo

```sh
dnf -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
dnf -y install https://rpms.remirepo.net/enterprise/remi-release-8.rpm
```

## Install PHP 

Install epel and remi repo before PHP installation.

Enable PHP 7.2 module first:

```sh
dnf module list php
dnf -y module reset php
dnf -y module enable php:7.2
```

Install php:
```sh
dnf -y install php
dnf -y install php-{common,cli,fpm,ctype,bcmath,pdo,mysqlnd,gd,mbstring,json,xml,pear,opcache,zip}
dnf -y install php-{pdo_mysql,filter,zlib,curl,iconv}
```

Note: openssl and tokenizer php extensions are included in php-common.

## Web Server Integration

Create `www` user and group:
```sh
/usr/sbin/groupadd www
/usr/sbin/useradd -g www -s /sbin/nologin www
```

Create necessary dirs:
```sh
mkdir -p /data/log/{php,nginx}
mkdir -p /data/www
chown www:www /data/www
```

nginx global config, especially user, log format, and log location:
```sh
mv /etc/nginx/nginx.conf{,_bak}
cat > /etc/nginx/nginx.conf <<'EOF'
user www www;
worker_processes  4;

error_log  /data/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] $host "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" $upstream_addr $request_time $upstream_response_time';

    access_log  /data/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
EOF
```

nginx server config:
```sh
mv /etc/nginx/conf.d/default.conf{,_bak}
cat > /etc/nginx/conf.d/default.conf <<'EOF'
server {
    listen       80;
    server_name  localhost;
    index        index.html index.php;
    root         /data/www;

    access_log  /data/log/nginx/access.log  main;

    location ~ \.php$ {
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }
}
EOF
```

Config PHP and php-fpm:
```sh
sed -i -e '/^user = apache/c\user = www' -e '/^group = apache/c\group = www' /etc/php-fpm.d/www.conf
sed -i '/^listen =/s!^.*$!listen = 9000!' /etc/php-fpm.d/www.conf
sed -i '/^php_admin_value\[error_log\]/s!/var/log/.*$!/data/log/php/php-fpm-www-error.log!' /etc/php-fpm.d/www.conf
sed -i '/session.save_path/s!/var/lib/php/session!/tmp!' /etc/php-fpm.d/www.conf

sed -i '/^opcache.enable=1/c\opcache.enable=0' /etc/php.d/10-opcache.ini

sed -i '/^;cgi.fix_pathinfo=/c\cgi.fix_pathinfo=0' /etc/php.ini
```

Enable and start nginx and php-fpm service:
```sh
systemctl enable nginx.service
systemctl start nginx.service
systemctl enable php-fpm.service
systemctl start php-fpm.service
```

Test HTML and PHP pages:

```sh
cat > /data/www/test.html <<'EOF'
<h1>It works.</h1>
EOF

cat > /data/www/phpinfo.php <<'EOF'
<?php phpinfo();
EOF

chown www:www /data/www/*
```


## Install MySQL 8.0

```sh
dnf module list mysql
dnf -y install mysql-server
```

Create necessary dirs:
```sh
mkdir -p /data/mysql/data
chown -R mysql:mysql /data/mysql
mkdir -p /data/binlog
chown mysql:mysql /data/binlog
```

my.cnf

```sh
cp -a /etc/my.cnf.d/mysql-server.cnf{,_bak}
cat > /etc/my.cnf.d/mysql-server.cnf <<'EOF'
[mysqld]
ssl       = 0
server-id = 1

datadir   = /data/mysql/data
log-error = /data/mysql/mysqld.log
pid-file  = /var/run/mysqld/mysqld.pid
socket    = /var/lib/mysql/mysql.sock

# common mysqld setting
read_only                 = 0
default_password_lifetime = 0
max_allowed_packet        = 16M   # same to master
user                      = mysql

# common InnoDB/XtraDB settings
# innodb_buffer_pool_instances  = 4      # default=8
innodb_buffer_pool_size        = 32768M  # x 1.2 + 2GB for OS = 16.4GB node w/o MyISAM
innodb_log_file_size           = 256M
innodb_flush_log_at_trx_commit = 2
innodb_flush_method            = O_DIRECT
innodb_file_per_table          = 1
default-storage-engine         = innodb

# common business settings
back_log             = 500
max_connections      = 3000
max_connect_errors   = 100000
thread_cache_size    = 64
table_open_cache     = 1024
sort_buffer_size     = 2M
read_buffer_size     = 2M
join_buffer_size     = 2M
read_rnd_buffer_size = 4M
open_files_limit     = 65535

# GTID
gtid_mode                = ON
enforce-gtid-consistency = ON

explicit_defaults_for_timestamp = ON

skip-external-locking
skip-name-resolve
skip-slave-start     = 1
character-set-server = utf8mb4 
collation-server     = utf8mb4_general_ci
EOF
```

Enable binlog if necessary:
```sh
cat >> /etc/my.cnf.d/mysql-server.cnf <<'EOF'

#binlog
log-bin             = mysql-bin.log
sync_binlog         = 1
relay-log           = relay-bin.log
relay-log-purge     = 0
log-slave-updates
sync_master_info    = 1
sync_relay_log      = 1
sync_relay_log_info = 1

#replication
replicate-same-server-id    = 0
log_bin                     = /data/binlog/mysql-bin
binlog-ignore-db            = mysql
binlog-ignore-db            = test
binlog-ignore-db            = information_schema
binlog-ignore-db            = performance_schema
replicate-ignore-db         = mysql
replicate-ignore-db         = test
replicate-ignore-db         = information_schema
replicate-ignore-db         = performance_schema
master_info_repository      = TABLE
relay_log_info_repository   = TABLE
slave_parallel_workers      = 0
slave_preserve_commit_order = 1
slave_parallel_type         = LOGICAL_CLOCK
EOF
```

Start MySQL:
```sh
systemctl enable mysqld.service
systemctl start mysqld.service
```

Secure MySQL installation run:
```sh
/usr/bin/mysql_secure_installation
```

## Install Redis

```sh
dnf module list redis
dnf -y module enable redis:remi-6.2
dnf -y install redis
systemctl enable redis.service
systemctl start redis.service
```

## Install MongoDB

```sh
cat > /etc/yum.repos.d/mongodb-org-5.0.repo <<'EOF'
[mongodb-org-5.0]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/5.0/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-5.0.asc
EOF

dnf -y install mongodb-org


```

## Other service or software

### PHP Composer

```sh
curl -sS https://getcomposer.org/installer | php
mv composer.phar /usr/local/bin/composer
```

### node.js

```sh
dnf -y module install nodejs:14
```

### phpMyAdmin

```sh
cd /data/www
wget https://files.phpmyadmin.net/phpMyAdmin/5.1.0/phpMyAdmin-5.1.0-all-languages.tar.xz
tar xJf phpMyAdmin-*.tar.gz
mv phpMyAdmin-*/ phpmyadmin/

chown -R www:www phpmyadmin/
cp -a phpmyadmin/config.sample.inc.php phpmyadmin/config.inc.php
```

Change `blowfish_secret`, `auth_type`, `host`, and other configrations.

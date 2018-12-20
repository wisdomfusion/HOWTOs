# How to Install nginx with PHP 7 and MySQL 5.7 Using Yum on CentOS 7

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
UUID=ee9868dc-bdab-46ed-8c08-92e12f26d765
DEVICE=ens33
ONBOOT=yes
IPADDR=192.168.10.10
PREFIX=24
GATEWAY=192.168.10.2
DNS1=202.106.0.20
DNS2=8.8.8.8
IPV6_PRIVACY=no

systemctl restart network.service
```

## hostname

```sh
hostnamectl set-hostname centos.example.com
```

## timezone

```sh
timedatectl list-timezones
timedatectl set-timezone Asia/Shanghai

# OR

ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

## ntpd

```sh
yum -y install ntp
systemctl enable ntpd.service
systemctl start ntpd.service
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

## keep systemd logs

```sh
mkdir -p /var/log/journal
systemctl restart systemd-journald.service
```

## Vim and other tools

```sh
yum -y install git man wget unzip zip net-tools nmap iptraf iotop htop sysstat bash-completion lrzsz
```

**simple Vim configs**

```sh
yum -y install vim-enhanced
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
yum -y groupinstall "Development Tools"
```

## Install nginx

```sh
cat > /etc/yum.repos.d/nginx.repo <<\EOF
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/7/$basearch/
gpgcheck=0
enabled=1
EOF

yum -y install nginx
```

## Install PHP 7

Install PHP from remi repository: https://rpms.remirepo.net/

```sh
rpm -ivUh https://rpms.remirepo.net/enterprise/remi-release-7.rpm
yum -y install yum-utils
yum-config-manager --enable remi-php71
yum -y install php-fpm php-bcmath php-cli php-ctype php-gd php-json php-mbstring php-mcrypt php-mysqlnd php-opcache php-openssl php-nette-tokenizer php-pdo php-xml php-xmlrpc php-pecl-imagick php-pecl-zip
```

## Web Server Integration

Create `www` user and group:
```sh
/usr/sbin/groupadd www
/usr/sbin/useradd -g www -s /sbin/nologin www
```

Create necessary dirs:
```sh
mkdir -p /data/logs/{php,nginx}
mkdir -p /data/www
chown www:www /data/www
```

nginx global config, especially user, log format, and log location:
```sh
mv /etc/nginx/nginx.conf{,_bak}
cat > /etc/nginx/nginx.conf <<'EOF'
user www www;
worker_processes  4;

error_log  /data/logs/nginx/error.log warn;
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

    access_log  /data/logs/nginx/access.log  main;

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

    access_log  /data/logs/nginx/access.log  main;

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
sed -i '/^php_admin_value\[error_log\]/s!/var/log/.*$!/data/logs/php/php-fpm-www-error.log!' /etc/php-fpm.d/www.conf
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


## Install Percona Server for MySQL 5.7

```sh
rpm -ivUh https://mirrors.tuna.tsinghua.edu.cn/percona/release/percona-release-0.1-7.noarch.rpm
cp -a /etc/yum.repos.d/percona-release.repo{,_bak}
sed -i 's!http://repo.percona.com/release/!https://mirrors.tuna.tsinghua.edu.cn/percona/release/!g' /etc/yum.repos.d/percona-release.repo
yum makecache
yum -y install Percona-Server-server-57
```

Create necessary dirs:
```sh
mkdir -p /data/mysql/data
touch /data/mysql/mysql-error.log
chown -R mysql:mysql /data/mysql
mkdir -p /data/binlogs
chown mysql:mysql /data/binlogs
```

my.cnf

```sh
cp -a /etc/my.cnf{,_bak}
cat > /etc/my.cnf <<'EOF'
[mysql]
default-character-set = utf8mb4
[mysqladmin]
default-character-set = utf8mb4
[mysqlcheck]
default-character-set = utf8mb4
[mysqldump]
default-character-set = utf8mb4
[mysqlimport]
default-character-set = utf8mb4
[mysqlshow]
default-character-set = utf8mb4

[mysqld_safe]
log-error=/var/log/mariadb/mariadb.log
pid-file=/var/run/mariadb/mariadb.pid

[mysqld]
ssl=0
server-id = 1

datadir=/data/mysql
log-error=/data/mysql/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
socket=/var/lib/mysql/mysql.sock

#common InnoDB/XtraDB settings
#innodb_buffer_pool_instances=4(default=8)
innodb_buffer_pool_size = 32768M  # x 1.2 + 2GB for OS = 16.4GB node w/o MyISAM
innodb_log_file_size = 256M  # suitable for most environments
innodb_flush_log_at_trx_commit = 2
innodb_flush_method = O_DIRECT
innodb_file_per_table = 1
default-storage-engine = innodb

#Percona Server enhancements
#http://www.percona.com/doc/percona-server
innodb_empty_free_list_algorithm = backoff
innodb_buffer_pool_dump_pct=30          #5.7 new
innodb_buffer_pool_load_at_startup = 1  #5.7 new
innodb_buffer_pool_dump_at_shutdown = 1 #5.7 new
relay_log_recovery = 1
innodb_corrupt_table_action = warn  # 5.5.10-20.1 introduced
log_slow_verbosity = full
userstat = ON  # 5.5.10-20.1 introduced
kill_idle_transaction = 5

#common business settings
back_log = 500
max_connections = 3000  # should be easy job in a big server
max_connect_errors = 100000
thread_cache_size = 64
table_open_cache = 1024
sort_buffer_size = 2M
read_buffer_size = 2M
join_buffer_size = 2M
read_rnd_buffer_size = 4M
open_files_limit = 65535

#GTID
gtid_mode = ON
enforce-gtid-consistency = ON

explicit_defaults_for_timestamp = ON

#common mysqld setting
skip-symbolic-links
read_only = 0
default_password_lifetime=0
max_allowed_packet = 16M  # same to master
user = mysql
skip-external-locking  # a.k.a skip-locking
skip-name-resolve
skip-slave-start = 1
character-set-server = utf8mb4  # default-character-set is deprecated in 5.0
collation-server = utf8mb4_general_ci
tmpdir = /dev/shm
log_output = FILE
general_log = OFF
slow_query_log = 1  # ON is not recognized in 5.1.46
binlog_format=ROW
slave_skip_errors = 1062
max_slowlog_size=5
max_slowlog_size=2097152 #percona 5.7 new
long_query_time = 1  # in seconds, determine slow query
general_log_file = query.log  # log is deprecated as of 5.1.29
slow_query_log_file = slow-query.log  # log_slow_queries and log_queries_not_using_index are deprecated as of 5.1.29
EOF
```

Enable binlog if necessary:
```sh
cat >> /opt/mysql/my.cnf <<'EOF'

#binlog
log-bin = mysql-bin.log
sync_binlog = 1  # BBU-backed RAID or flash
relay-log = relay-bin.log  # auto purge by default, see relay-log-purge
relay-log-purge = 0  # MHA node
log-slave-updates
sync_master_info               = 1
sync_relay_log                 = 1
sync_relay_log_info            = 1

#replication
replicate-same-server-id = 0
log_bin = /data/binlogs/mysql-bin
binlog-ignore-db = mysql
binlog-ignore-db = test
binlog-ignore-db = information_schema
binlog-ignore-db = performance_schema
replicate-ignore-db = mysql
replicate-ignore-db = test
replicate-ignore-db = information_schema
replicate-ignore-db = performance_schema
master_info_repository=TABLE
relay_log_info_repository=TABLE
slave_parallel_workers=0
slave_preserve_commit_order=1
slave_parallel_type = LOGICAL_CLOCK
EOF
```

Start MySQL:
```sh
systemctl enable mysqld.service
systemctl start mysqld.service
```

Find Percona MySQL Default root password:
```sh
grep "temporary password" /data/mysql/mysqld.log
```

Secure MySQL installation run:
```sh
/usr/bin/mysql_secure_installation
```

## Install Redis

```sh
yum -y install redis
systemctl enable redis.service
systemctl start redis.service
```

## Other service or software

### PHP Composer

```sh
curl -sS https://getcomposer.org/installer | php
mv composer.phar /usr/local/bin/composer
```

### node.js

```sh
curl --silent --location https://rpm.nodesource.com/setup_10.x | bash -
yum -y install nodejs
```

### phpMyAdmin

```sh
cd /data/www
wget https://files.phpmyadmin.net/phpMyAdmin/4.8.3/phpMyAdmin-4.8.3-all-languages.tar.gz
tar zxvf phpMyAdmin-4.8.3-all-languages.tar.gz
mv phpMyAdmin-*/ phpmyadmin/

chown -R www:www phpmyadmin/
cp -a phpmyadmin/config.sample.inc.php phpmyadmin/config.inc.php
```

Change `blowfish_secret`, `auth_type`, `host`, and other configs.

### Python 3.6

```sh
yum -y update
yum -y install yum-utils
rpm -ivUh https://centos7.iuscommunity.org/ius-release.rpm
yum -y install python36u python36u-pip python36u-devel
```

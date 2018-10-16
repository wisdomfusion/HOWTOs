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
yum install -y ntp
systemctl enable ntpd.service
systemctl start ntpd.service
```

## disable SELinux

```sh
setenforce 0
sed -i 's/^SELINUX=.*$/SELINUX=disabled/' /etc/selinux/config
```

reboot and confirm the status:
```sh
# sestatus
SELinux status:                 disabled
```

## open file limit

```sh
ulimit -n 65535
sed -i '/DefaultLimitNOFILE/c\DefaultLimitNOFILE=65535' /etc/systemd/system.conf
```

## keep systemd logs

```sh
mkdir -p /var/log/journal
systemctl restart systemd-journald.service
```

## Vim and other tools

```sh
yum -y install man wget unzip zip net-tools nmap iptraf iotop htop sysstat ntp vim-enhanced bash-completion lrzsz
```

**simple Vim configs**

```sh
alias vi='vim'
echo "alias vi='vim'" >> ~/.bashrc
echo "set fencs=utf-8,ucs-bom,gb18030,gbk,gb2312,cp936" >> ~/.vimrc
echo "set fileformats=unix,dos" >> ~/.vimrc
```

## epel

```sh
yum -y install epel-release
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

use webtatic binary
https://webtatic.com/

```sh
rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
yum -y install php71w-cli php71w-fpm php71w-mysqlnd php71w-pdo php71w-opcache php71w-mbstring php71w-mcrypt php71w-gd php71w-pecl-imagick php71w-pecl-redis php71w-pecl-memcached php71w-pecl-mongodb php71w-xml php71w-xmlrpc php71w-process php71w-bcmath
```

## Web Server Integration

```sh
/usr/sbin/groupadd www
/usr/sbin/useradd -g www -s /sbin/nologin www

cat > /etc/nginx/conf.d/default.conf <<'EOF'
server {
    listen       80;
    server_name  localhost;
    index        index.html index.php;
    root         /web/www/;

    access_log  /web/logs/nginx/access.log  main;

    location ~ \.php$ {
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }
}
EOF
```


## Install Percona Server for MySQL 5.7

```sh
cd /opt
wget https://www.percona.com/downloads/Percona-Server-LATEST/Percona-Server-5.7.23-23/binary/tarball/Percona-Server-5.7.23-23-Linux.x86_64.ssl101.tar.gz
tar zxvf Percona-Server-5.7.23-23-Linux.x86_64.ssl101.tar.gz -C /opt/mysql
```

my.cnf

```sh
cat > /opt/mysql/my.cnf <<'EOF'
!include .my.cnf

# only certain clients support default-character-set
[mysql]
prompt = 'mysql \u@[\h:\p \d] > '
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
log-error = /web/mysql/mysql-error.log

[mysqld]
ssl=0
server-id = 1

# chain specific settings
port = 3306
socket = /tmp/mysql.sock
basedir = /opt/mysql
datadir = /web/mysql/data
pid-file = /web/mysql/data/mysql.pid

# common InnoDB/XtraDB settings
#innodb_buffer_pool_instances=4(default=8)
innodb_buffer_pool_size = 32768M  # x 1.2 + 2GB for OS = 16.4GB node w/o MyISAM
innodb_log_file_size = 256M  # suitable for most environments
innodb_flush_log_at_trx_commit = 2
innodb_flush_method = O_DIRECT
innodb_file_per_table = 1
default-storage-engine = innodb

# enable gtid mode

# Percona Server enhancements
#  http://www.percona.com/doc/percona-server
innodb_empty_free_list_algorithm = backoff
innodb_buffer_pool_dump_pct=30          #5.7 new
innodb_buffer_pool_load_at_startup = 1  #5.7 new
innodb_buffer_pool_dump_at_shutdown = 1 #5.7 new
relay_log_recovery = 1
innodb_corrupt_table_action = warn  # 5.5.10-20.1 introduced
log_slow_verbosity = full
userstat = ON  # 5.5.10-20.1 introduced
kill_idle_transaction = 5

# common business settings
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

# common mysqld setting
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
log_bin = /web/logs/binlogs/mysql-bin
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

setup MySQL

```sh
cp -a /opt/mysql/support-files/mysql.server /web/mysql/mysql
sed -i 's/^\(basedir=\).*$/\1\/opt\/mysql/' /web/mysql/mysql
sed -i 's/^\(datadir=\).*$/\1\/web\/mysql\/data/' /web/mysql/mysql

/opt/mysql/bin/mysqld --initialize-insecure --user=mysql --basedir=/opt/mysql --datadir=/web/mysql/data

/web/mysql/mysql start

DBROOTPWD=123456
/opt/mysql/bin/mysql -e "grant all privileges on *.* to root@'127.0.0.1' identified by \"$DBROOTPWD\" with grant option;"
/opt/mysql/bin/mysql -e "grant all privileges on *.* to root@'localhost' identified by \"$DBROOTPWD\" with grant option;"
```

add MySQL commands to PATH

```sh
export PATH=$PATH:/opt/mysql/bin
echo 'export PATH=$PATH:/opt/mysql/bin' >> ~/.bashrc
```

## Install Redis

```sh
yum install -y redis
systemctl start redis.service
```

Confirm Redis installed successfully:

```sh
# redis-cli ping
PONG
```

## Other service or software

### PHP Composer

```sh
curl -sS https://getcomposer.org/installer | php
mv composer.phar /usr/local/bin/composer
```

### Git

```sh
yum install -y git
```

### node.js

```sh
curl --silent --location https://rpm.nodesource.com/setup_8.x | bash -
yum -y install nodejs
```

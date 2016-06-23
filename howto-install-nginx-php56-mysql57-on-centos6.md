# How to Install nginx with PHP 5.6 and MySQL 5.7 on CentOS 6

## 0. 约定

- 暂用系统用户 root 方便安装调试
- 主机IP地址：192.168.1.11
- 软件包下载到 `/usr/src/` 目录下
- 所有软件包安装在 `/usr/local/webserver/` 目录下
- `/u01/mysql/` 存放数据库
- `/u01/www/` 存放网站数据
- `/u01/logfiles/` 存放网站日志

## 1. 系统要求

系统环境：

- CentOS 6.8

所需软件包：

    nginx-1.10.0.tar.gz
    php-5.6.22.tar.gz
    mysql-5.7.13.tar.gz

相关库：

    boost_1_59_0.tar.gz
    libiconv-1.14.tar.gz
    libmcrypt-2.5.8.tar.gz
    mcrypt-2.6.8.tar.gz
    memcache-2.2.7.tgz
    mhash-0.9.9.9.tar.gz
    pcre-8.38.tar.gz
    ImageMagick.tar.gz
    imagick-3.4.1.tgz
    
其他服务软件包：

    memcached-1.4.25.tar.gz
    redis-3.2.0.tar.gz

## 2. 系统基本设定

### 2.1. 网络设置

网络设置：

```sh
cat /etc/sysconfig/network-scripts/ifcfg-eth0
DEVICE=eth0
TYPE=Ethernet
ONBOOT=yes
BOOTPROTO=static
IPADDR=192.168.1.11
NETMASK=255.255.255.0
GATEWAY=192.168.1.1
```

DNS 服务器：

```sh
echo "nameserver 202.106.196.115" >> /etc/resolv.conf
echo "nameserver 114.114.114.114" >> /etc/resolv.conf
```

### 2.2. 禁用 SELinux

```sh
setenforce 0
sed -i 's/^SELINUX=*$/SELINUX=disabled/' /etc/selinux/config
```

### 2.3. 禁用 IPv6
```sh
sysctl -w net.ipv6.conf.all.disable_ipv6=1
sysctl -w net.ipv6.conf.default.disable_ipv6=1
```

### 2.4. 进程打开文件数
```sh
echo >> /etc/security/limits.conf
echo "* soft nofile 65535" >> /etc/security/limits.conf
echo "* hard nofile 65535" >> /etc/security/limits.conf
```

## 3. 安装前的准备

### 3.1. 编译工具及相关库

```sh
sudo -s
LANG=C
yum -y install gcc gcc-c++ autoconf automake cmake zlib zlib-devel compat-libstdc++-33 glibc glibc-devel glib2 glib2-devel bzip2 bzip2-devel unzip zip nmap ncurses ncurses-devel sysstat ntp curl curl-devel libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel libtiff-devel gd gd-devel libxml2 libxml2-devel libXpm libXpm-devel libmcrypt libmcrypt-devel krb5 krb5-devel libidn libidn-devel openssl openssl-devel openldap openldap-devel nss_ldap openldap-clients openldap-servers pam-devel libicu libicu-devel
```

### 3.2. 安装常用工具

```sh
yum -y install man wget iptraf iotop vim-enhanced lrzsz

alias vi='vim'
echo "alias vi='vim'" >> ~/.bashrc
echo "set fencs=utf-8,ucs-bom,gb18030,gbk,gb2312,cp936" >> ~/.vimrc
echo "set fileformats=unix,dos" >> ~/.vimrc
```

### 3.3. 下载所需软件包

```sh
cd /usr/src
wget http://nginx.org/download/nginx-1.10.0.tar.gz
wget http://cn2.php.net/distributions/php-5.6.22.tar.gz
wget http://cn2.php.net/distributions/php-7.0.7.tar.gz
wget http://cdn.mysql.com//Downloads/MySQL-5.7/mysql-5.7.13.tar.gz
wget http://downloads.sourceforge.net/project/boost/boost/1.59.0/boost_1_59_0.tar.gz
wget http://ftp.gnu.org/gnu/libiconv/libiconv-1.14.tar.gz
wget http://downloads.sourceforge.net/mcrypt/libmcrypt-2.5.8.tar.gz
wget http://downloads.sourceforge.net/mcrypt/mcrypt-2.6.8.tar.gz
wget http://pecl.php.net/get/memcache-2.2.7.tgz
wget http://downloads.sourceforge.net/mhash/mhash-0.9.9.9.tar.gz
wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.38.tar.gz
wget http://www.imagemagick.org/download/ImageMagick.tar.gz
wget https://pecl.php.net/get/imagick-3.4.1.tgz
```

## 4. 安装


### 4.1. 相关库

```sh
tar zxvf libiconv-1.14.tar.gz
cd libiconv-1.14
./configure --prefix=/usr/local
make && make install
cd ../

tar zxvf libmcrypt-2.5.8.tar.gz
cd libmcrypt-2.5.8/
./configure && make && make install
/sbin/ldconfig
cd libltdl/
./configure --enable-ltdl-install
make && make install
cd ../../

tar zxvf mhash-0.9.9.9.tar.gz
cd mhash-0.9.9.9/
./configure && make && make install
cd ../

ln -s /usr/local/lib/libmcrypt.la /usr/lib/libmcrypt.la
ln -s /usr/local/lib/libmcrypt.so /usr/lib/libmcrypt.so
ln -s /usr/local/lib/libmcrypt.so.4 /usr/lib/libmcrypt.so.4
ln -s /usr/local/lib/libmcrypt.so.4.4.8 /usr/lib/libmcrypt.so.4.4.8
ln -s /usr/local/lib/libmhash.a /usr/lib/libmhash.a
ln -s /usr/local/lib/libmhash.la /usr/lib/libmhash.la
ln -s /usr/local/lib/libmhash.so /usr/lib/libmhash.so
ln -s /usr/local/lib/libmhash.so.2 /usr/lib/libmhash.so.2
ln -s /usr/local/lib/libmhash.so.2.0.1 /usr/lib/libmhash.so.2.0.1
ln -s /usr/local/bin/libmcrypt-config /usr/bin/libmcrypt-config

tar zxvf mcrypt-2.6.8.tar.gz
cd mcrypt-2.6.8/
/sbin/ldconfig
./configure && make && make install
cd ../

echo "/usr/local/lib" >> /etc/ld.so.conf
/sbin/ldconfig
```

### 4.2. 安装 MySQL 5.7

MySQL 5.7 GA版本的发布，也就是说从现在开始5.7已经可以在生产环境中使用，有任何问题官方都将立刻修复。

MySQL 5.7 主要特性：
- 更好的性能：对于多核 CPU、固态硬盘、锁有着更好的优化，每秒 100W QPS 已不再是 MySQL 的追求，下个版本能否上 200W QPS才 是吾等用户更关心的
- 更好的 InnoDB 存储引擎
- 更为健壮的复制功能：复制带来了数据完全不丢失的方案，传统金融客户也可以选择使用 MySQL 数据库。此外，GTID 在线平滑升级也变得可能
- 更好的优化器：优化器代码重构的意义将在这个版本及以后的版本中带来巨大的改进，Oracle 官方正在解决 MySQL 之前最大的难题
- 原生 JSON 类型的支持
- 更好的地理信息服务支持：InnoDB 原生支持地理位置类型，支持 GeoJSON，GeoHash 特性
- 新增 sys 库：以后这会是 DBA 访问最频繁的库

MySQL 编译安装时需要用到 boost C++ 库：
```sh
cd /usr/src/
tar zxvf boost_1_59_0.tar.gz -C /usr/local/
```

正式安装前先建立 `mysql:mysql` 用户：

```sh
/usr/sbin/groupadd mysql
/usr/sbin/useradd -g mysql mysql
```

解压前配置软件包

```sh
tar zxvf mysql-5.7.13.tar.gz
cd mysql-5.7.13/
cmake \
-DCMAKE_INSTALL_PREFIX=/usr/local/webserver/mysql \
-DMYSQL_DATADIR=/u01/mysql \
-DEXTRA_CHARSETS=all \
-DDEFAULT_CHARSET=utf8 \
-DDEFAULT_COLLATION=utf8_general_ci \
-DMYSQL_TCP_PORT=3306 \
-DMYSQL_USER=mysql \
-DSYSCONFDIR=/etc \
-DWITH_MYISAM_STORAGE_ENGINE=1 \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_PARTITION_STORAGE_ENGINE=1 \
-DWITH_ARCHIVE_STORAGE_ENGINE=1 \
-DWITH_MEMORY_STORAGE_ENGINE=1 \
-DWITH_BOOST=/usr/local/boost_1_59_0 \
-DENABLE_DOWNLOADS=1
```

如果主要用于移动端数据存储，可以默认 utf8mb4 编码，可存储 emoji 表情符号：

```sh
-DDEFAULT_CHARSET=utf8mb4 \
-DDEFAULT_COLLATION=utf8mb4_general_ci \
```

编译安装：
```sh
make -j `nproc`
make install
```

MySQL 启动脚本：

```sh
cp /usr/local/webserver/mysql/support-files/mysql.server /etc/init.d/mysqld
chmod +x /etc/init.d/mysqld
chkconfig --add mysqld
chkconfig mysqld on
```

必要的配置：

```sh
cat > /etc/my.cnf <<\EOF
[client]
port = 3306
socket = /tmp/mysql.sock
#default-character-set = utf8mb4

[mysqld]
port = 3306
socket = /tmp/mysql.sock
basedir = /usr/local/webserver/mysql
datadir = /u01/mysql
pid-file = /u01/mysql/mysql.pid
user = mysql
bind-address = 0.0.0.0
server-id = 1
#init-connect = 'SET NAMES utf8mb4'
#character-set-server = utf8mb4
#skip-name-resolve
#skip-networking

back_log = 300
max_connections = 1000
max_connect_errors = 6000
open_files_limit = 65535
table_open_cache = 128
max_allowed_packet = 4M
binlog_cache_size = 1M
max_heap_table_size = 8M
tmp_table_size = 16M
read_buffer_size = 2M
read_rnd_buffer_size = 8M
sort_buffer_size = 8M
join_buffer_size = 8M
key_buffer_size = 4M
thread_cache_size = 8
query_cache_type = 1
query_cache_size = 8M
query_cache_limit = 2M
ft_min_word_len = 4
log_bin = mysql-bin
binlog_format = mixed
expire_logs_days = 30
log_error = /u01/mysql/mysql-error.log
slow_query_log = 1
long_query_time = 1
slow_query_log_file = /u01/mysql/mysql-slow.log
performance_schema = 0
explicit_defaults_for_timestamp
#lower_case_table_names = 1
skip-external-locking

default_storage_engine = InnoDB
#default-storage-engine = MyISAM
innodb_file_per_table = 1
innodb_open_files = 500
innodb_buffer_pool_size = 64M
innodb_write_io_threads = 4
innodb_read_io_threads = 4
innodb_thread_concurrency = 0
innodb_purge_threads = 1
innodb_flush_log_at_trx_commit = 2
innodb_log_buffer_size = 2M
innodb_log_file_size = 32M
innodb_log_files_in_group = 3
innodb_max_dirty_pages_pct = 90
innodb_lock_wait_timeout = 120

bulk_insert_buffer_size = 8M
myisam_sort_buffer_size = 8M
myisam_max_sort_file_size = 10G
myisam_repair_threads = 1
interactive_timeout = 28800
wait_timeout = 28800

[mysqldump]
quick
max_allowed_packet = 16M

[myisamchk]
key_buffer_size = 8M
sort_buffer_size = 8M
read_buffer = 4M
write_buffer = 4M
EOF
```

初始化数据库，请保持 `/u01/mysql/` 目录为空：
```sh
mkdir -p /u01/mysql
chown mysql:mysql /u01/mysql/
/usr/local/webserver/mysql/bin/mysqld --initialize-insecure --user=mysql --basedir=/usr/local/webserver/mysql --datadir=/u01/mysql
```

启动 MySQL：

```sh
/etc/init.d/mysqld start
```

给 MySQL root 用户添加密码：

```sh
dbrootpwd=123456
/usr/local/webserver/mysql/bin/mysql -e "grant all privileges on *.* to root@'127.0.0.1' identified by \"$dbrootpwd\" with grant option;"
/usr/local/webserver/mysql/bin/mysql -e "grant all privileges on *.* to root@'localhost' identified by \"$dbrootpwd\" with grant option;"
```

关闭 MySQL：

```sh
/etc/init.d/mysqld stop
```

### 4.3. 安装 PHP 5.6.22

先建立 www:www 用户：

```sh
/usr/sbin/groupadd www
/usr/sbin/useradd -g www www
mkdir -p /u01/www
chmod +w /u01/www
chown -R www:www /u01/www
```

安装 PHP：

```sh
cd /usr/src/
tar zxvf php-5.6.22.tar.gz
cd php-5.6.22/
./configure --prefix=/usr/local/webserver/php \
--with-config-file-path=/usr/local/webserver/php/etc \
--with-libdir=lib64 \
--enable-fpm \
--with-fpm-user=www \
--with-fpm-group=www \
--enable-mysqlnd \
--with-mysql=mysqlnd \
--with-mysqli=mysqlnd \
--with-pdo-mysql=mysqlnd \
--enable-opcache \
--enable-pcntl \
--enable-mbstring \
--enable-soap \
--enable-zip \
--enable-calendar \
--enable-bcmath \
--enable-exif \
--enable-ftp \
--enable-intl \
--enable-xml \
--enable-sockets \
--with-xmlrpc \
--with-openssl \
--with-zlib \
--enable-mbregex \
--with-curl \
--with-gd \
--with-zlib-dir=/usr/lib \
--with-png-dir=/usr/lib \
--with-jpeg-dir=/usr/lib \
--with-freetype-dir=/usr/include/freetype2/ \
--with-gettext \
--with-mhash \
--with-mcrypt \
--with-ldap \
--disable-ipv6
```

**注意：**PHP 默认配置文件存放在 `php/lib/` 下，为什么不放在 `php/etc/` 下？为什么不放在 `php/etc/` 下？为什么？好，我把它改到 `php/etc/` 下，所以上面编译选项中我添加了这一项 `--with-config-file-path=/usr/local/webserver/php/etc`。如果你觉得默认就很爽，那你大可以把这项去掉，能很欢快地找到 php.ini 并使之生效即可，如果发现配置不生效，请用 `php --ini` 命令 check 一下。

编译安装：

```sh
make ZEND_EXTRA_LIBS='-liconv' -j `nproc`
make install
```

php.ini 和 php-fpm.conf

```sh
cp php.ini-production /usr/local/webserver/php/etc/php.ini
cp /usr/local/webserver/php/etc/php-fpm.conf.default /usr/local/webserver/php/etc/php-fpm.conf
```

php-fpm.conf 配置 `vi /usr/local/webserver/php/etc/php-fpm.conf`：

    [global]
    pid = run/php-fpm.pid
    error_log = log/php-fpm.log
    emergency_restart_threshold = 60
    emergency_restart_interval = 60
    process_control_timeout = 5s
    daemonize = yes
    rlimit_files = 65535

    [www]
    user = www
    group = www
    pm = static
    pm.max_children = 384
    pm.start_servers = 20
    pm.min_spare_servers = 5
    pm.max_spare_servers = 35
    pm.process_idle_timeout = 10s
    pm.max_requests = 51200
    request_terminate_timeout = 0

可以直接使用如下系列命令达到如上修改效果：

```sh
sed -i '/^;pid/s/^;//' /usr/local/webserver/php/etc/php-fpm.conf
sed -i '/^;error_log/s/^;//' /usr/local/webserver/php/etc/php-fpm.conf
sed -i '/^;emergency_restart_threshold/c\emergency_restart_threshold = 60' /usr/local/webserver/php/etc/php-fpm.conf
sed -i '/^;emergency_restart_interval/c\emergency_restart_interval = 60' /usr/local/webserver/php/etc/php-fpm.conf
sed -i '/^;process/c\process_control_timeout = 5s' /usr/local/webserver/php/etc/php-fpm.conf
sed -i '/^;daemonize/s/^;//' /usr/local/webserver/php/etc/php-fpm.conf
sed -i '/^;rlimit_files/c\rlimit_files = 65535' /usr/local/webserver/php/etc/php-fpm.conf
sed -i '/^pm =/c\pm = static' /usr/local/webserver/php/etc/php-fpm.conf
sed -i '/^pm.max_children/s/[0-9]\+$/256/' /usr/local/webserver/php/etc/php-fpm.conf
sed -i '/^pm.start_servers/s/[0-9]\+$/20/' /usr/local/webserver/php/etc/php-fpm.conf
sed -i '/^pm.min_spare_servers/s/[0-9]\+$/5/' /usr/local/webserver/php/etc/php-fpm.conf
sed -i '/^pm.max_spare_servers/s/[0-9]\+$/35/' /usr/local/webserver/php/etc/php-fpm.conf
sed -i '/^;pm.process_idle_timeout/s/^;//' /usr/local/webserver/php/etc/php-fpm.conf
sed -i '/^;pm.max_requests/c\pm.max_requests = 51200' /usr/local/webserver/php/etc/php-fpm.conf
sed -i '/^;request_terminate_timeout/s/^;//' /usr/local/webserver/php/etc/php-fpm.conf
```

php-fpm 启动脚本：

```sh
cp sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm
chmod +x /etc/init.d/php-fpm
chkconfig --add php-fpm
chkconfig php-fpm on
```

PHP 模块安装：

```sh
cd /usr/src/
tar zxvf memcache-2.2.7.tgz
cd memcache-2.2.7/
/usr/local/webserver/php/bin/phpize
./configure --with-php-config=/usr/local/webserver/php/bin/php-config
make && make install
cd ../

tar zxvf ImageMagick.tar.gz
cd ImageMagick-*/
./configure && make && make install
/sbin/ldconfig
cd ../

tar zxvf imagick-3.4.1.tgz
cd imagick-3.4.1/
/sbin/ldconfig
/usr/local/webserver/php/bin/phpize
./configure --with-php-config=/usr/local/webserver/php/bin/php-config
make && make install
cd ../

wget https://pecl.php.net/get/redis-2.2.7.tgz
tar zxvf redis-2.2.7.tgz
cd redis-2.2.7
/usr/local/webserver/php/bin/phpize
./configure --with-php-config=/usr/local/webserver/php/bin/php-config
make && make install
cd ../
```

编译安装好模块，还要在 `php.ini` 里添加这些模块，使之生效：

```sh
vi /usr/local/webserver/php/etc/php.ini
```

配置项：

    ; extension_dir = "ext"
    ; 在该行下添加如下配置：
    extension_dir = "/usr/local/webserver/php/lib/php/extensions/no-debug-non-zts-20131226/"
    extension=imagick.so
    extension=memcache.so
    extension=redis.so

再次注意 `php.ini` 的位置，这个真的很重要！

还是在 `php.ini` 文件中，启用内置的 opcache 模块，并调整相关配置：

    ; 添加
    zend_extension=opcache.so
    ; 修改
    opcache.enable=1
    opcache.memory_consumption=256
    opcache.interned_strings_buffer=8
    opcache.max_accelerated_files=4000
    opcache.revalidate_freq=60
    opcache.fast_shutdown=1

重新启动 php-fpm 即可生效。

### 4.4. 安装 nginx 1.10.0

先要安装 PCRE 支持：

```sh
cd /usr/src/
tar zxvf pcre-8.38.tar.gz
cd pcre-8.38/
./configure && make && make install
cd ../
```

配置并安装 nginx：

```sh
tar zxvf nginx-1.10.0.tar.gz
cd nginx-1.10.0
/sbin/ldconfig
./configure \
--prefix=/usr/local/webserver/nginx \
--user=www \
--group=www \
--with-http_v2_module \
--with-http_sub_module \
--with-http_ssl_module \
--with-http_gunzip_module \
--with-http_gzip_static_module \
--with-http_realip_module \
--with-http_flv_module \
--with-http_stub_status_module \
--with-http_addition_module
make && make install
cd ../
```

启动脚本：

```sh
cat > /etc/init.d/nginx <<\EOF
#!/bin/sh
#
# nginx - this script starts and stops the nginx daemon
#
# chkconfig:   - 85 15
# description:  NGINX is an HTTP(S) server, HTTP(S) reverse \
#               proxy and IMAP/POP3 proxy server
# processname: nginx
# nginx:       /usr/local/webserver/nginx/sbin/nginx
# config:      /usr/local/webserver/nginx/conf/nginx.conf
# pidfile:     /usr/local/webserver/nginx/logs/nginx.pid

# Source function library.
. /etc/rc.d/init.d/functions

# Source networking configuration.
. /etc/sysconfig/network

# Check that networking is up.
[ "$NETWORKING" = "no" ] && exit 0

nginx="/usr/local/webserver/nginx/sbin/nginx"
prog=$(basename $nginx)

NGINX_CONF_FILE="/usr/local/webserver/nginx/conf/nginx.conf"

lockfile=/var/lock/subsys/nginx

make_dirs() {
   # make required directories
   user=`$nginx -V 2>&1 | grep "configure arguments:" | sed 's/[^*]*--user=\([^ ]*\).*/\1/g' -`
   if [ -z "`grep $user /etc/passwd`" ]; then
       useradd -M -s /bin/nologin $user
   fi
   options=`$nginx -V 2>&1 | grep 'configure arguments:'`
   for opt in $options; do
       if [ `echo $opt | grep '.*-temp-path'` ]; then
           value=`echo $opt | cut -d "=" -f 2`
           if [ ! -d "$value" ]; then
               # echo "creating" $value
               mkdir -p $value && chown -R $user $value
           fi
       fi
   done
}

start() {
    [ -x $nginx ] || exit 5
    [ -f $NGINX_CONF_FILE ] || exit 6
    make_dirs
    echo -n $"Starting $prog: "
    daemon $nginx -c $NGINX_CONF_FILE
    retval=$?
    echo
    [ $retval -eq 0 ] && touch $lockfile
    return $retval
}

stop() {
    echo -n $"Stopping $prog: "
    killproc $prog -QUIT
    retval=$?
    echo
    [ $retval -eq 0 ] && rm -f $lockfile
    return $retval
}

restart() {
    configtest || return $?
    stop
    sleep 1
    start
}

reload() {
    configtest || return $?
    echo -n $"Reloading $prog: "
    killproc $nginx -HUP
    RETVAL=$?
    echo
}

force_reload() {
    restart
}

configtest() {
  $nginx -t -c $NGINX_CONF_FILE
}

rh_status() {
    status $prog
}

rh_status_q() {
    rh_status >/dev/null 2>&1
}

case "$1" in
    start)
        rh_status_q && exit 0
        $1
        ;;
    stop)
        rh_status_q || exit 0
        $1
        ;;
    restart|configtest)
        $1
        ;;
    reload)
        rh_status_q || exit 7
        $1
        ;;
    force-reload)
        force_reload
        ;;
    status)
        rh_status
        ;;
    condrestart|try-restart)
        rh_status_q || exit 0
            ;;
    *)
        echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload|configtest}"
        exit 2
esac
EOF

chmod +x /etc/init.d/nginx
chkconfig --add nginx
chkconfig nginx on
```

相关配置：

```sh
mkdir -p /u01/logfiles/nginx
chmod +w /u01/logfiles/nginx
chown -R www:www /u01/logfiles/nginx

cp /usr/local/webserver/nginx/conf/nginx.conf{,.original}
cat > nginx.conf <<\EOF
user  www www;
worker_processes 1;
error_log  /u01/logfiles/nginx/nginx_error.log  crit;
#pid        logs/nginx.pid;

worker_rlimit_nofile 51200;

events
{
  use epoll;
  worker_connections 51200;
}

http
{
  include       mime.types;
  default_type  application/octet-stream;

  server_names_hash_bucket_size 128;
  client_header_buffer_size 32k;
  large_client_header_buffers 4 32k;
  client_max_body_size 8m;

  sendfile on;
  tcp_nopush     on;

  keepalive_timeout 5;

  tcp_nodelay on;

  fastcgi_connect_timeout 300;
  fastcgi_send_timeout 300;
  fastcgi_read_timeout 300;
  fastcgi_buffer_size 64k;
  fastcgi_buffers 8 64k;
  fastcgi_busy_buffers_size 128k;
  fastcgi_temp_file_write_size 128k;

  gzip on;
  gzip_min_length  1k;
  gzip_buffers     4 16k;
  gzip_http_version 1.0;
  gzip_comp_level 2;
  gzip_types       text/plain application/x-javascript text/css application/xml;
  gzip_vary on;

  log_format  access  '$remote_addr - $remote_user [$time_local] "$request" '
              '$status $body_bytes_sent "$http_referer" '
              '"$http_user_agent" $http_x_forwarded_for';

  server
  {
    listen       80;
    server_name  localhost;
    index index.php index.html;
    root  /u01/www/;

    location ~ \.php$ {
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        include        fastcgi.conf;
    }
    location ~ /\.ht {
        deny  all;
    }
    location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$
    {
      expires      30d;
    }
    location ~ .*\.(js|css)?$
    {
      expires      1h;
    }

    access_log  /u01/logfiles/nginx/localhost.access.log  access;
  }

}
EOF
```

nginx 日志轮询：

- nginx 日志存放位置：`/u01/logfiles/nginx`
- 日志轮询脚本：`/usr/local/webserver/nginx/cut_nginx_log.sh`
- nginx pid 文件位置：`/usr/local/webserver/nginx/logs/nginx.pid`

```sh
cat > /usr/local/webserver/nginx/cut_nginx_log.sh <<\EOF
#!/bin/bash
# cut_nginx_log.sh
# This script run at 00:00

# The Nginx logs path
logs_path="/u01/logfiles/nginx"
newlogs_path=${logs_path}/$(date -d 'yesterday' '+%Y%m%d')

mkdir -p ${newlogs_path}
mv ${logs_path}/*.log ${newlogs_path}

kill -USR1 `cat /usr/local/webserver/nginx/logs/nginx.pid`
EOF
```

crontab 中添加定时任务，00:00 执行日志轮询：

```sh
crontab -e
0 0 * * * /bin/bash /usr/local/webserver/nginx/cut_nginx_log.sh
```

## 5. 其他服务安装

### 5.1. sshd

添加管理用户

```sh
useradd -G wheel username
```

启用 wheel 用户组的 su 权限

vi /etc/pam.d/su 取消注释如下行：

    #auth required pam_wheel.so use_uid

```sh
echo "SU_WHEEL_ONLY yes" >> /etc/login.defs
```

SSH2 的公钥与私钥的建立

```sh
su - username
ssh-keygen -t rsa
cd ~/.ssh
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
rm -f ~/.ssh/id_rsa.pub
chmod 400 ~/.ssh/authorized_keys
```

vi /etc/ssh/sshd_config

    ServerKeyBits 1024
    PermitRootLogin no
    PasswordAuthentication no
    PermitEmptyPasswords no

### 5.2. rsync

```sh
yum -y install rsync xinetd
```

启用 rsync 服务：

vi /etc/xinetd.d/rsync

修改 disable = yes 为 disable = no

为 rsync 客户端添加用户名及密码：

vi /etc/rsyncd.secrets

    adminname:hispassword

rsync 配置文件的修改：

vi /etc/rsyncd.conf

    uid = root
    gid = root

    max connections = 10
    log file = /srvapp/logfiles/rsyncd.log
    timeout = 300

    [test]
    comment = test rsync
    path = /home/tmp
    read only = no
    list = yes
    hosts allow = 10.10.105.0/24
    auth users = g9N1lsEm8Q
    secrets file = /etc/rsyncd.secrets

rsync 安全相关配置：

```sh
chown root.root /etc/rsyncd.*
chmod 600 /etc/rsyncd.*
```

重启 xinetd 服务使用配置生效：

```sh
service xinetd restart
```

另外，rsync 服务使用873端口，iptables 中的端口设置要注意。

### 5.3. ProFTPd

FTP 服务器


## 6. 系统安全加固

### 6.1. iptables 防火墙

```sh
mkdir -p /usr/local/firewall
touch /usr/local/firewall/iptables.rule
touch /usr/local/firewall/iptalbes.allow
touch /usr/local/firewall/iptables.deny
chmod 700 /usr/local/firewall/iptables.*

cat > /usr/local/firewall/iptables.rule <<\EOF
#!/bin/bash

# 请先输入您的相关参数，不要输入错误了！
  EXTIF="eth0"              # 这个是可以连上 Public IP 的网络接口
  INIF=""               # 内部 LAN 的连接接口；若无请填 ""
  INNET=""    # 内部 LAN 的网域，如192.168.1.0/24，若没有内部 LAN 请设定为 ""
  export EXTIF INIF INNET

# 第一部份，针对本机的防火墙设定！###########################
# 1. 先设定好核心的网络功能：
  echo "1" > /proc/sys/net/ipv4/tcp_syncookies
  echo "1" > /proc/sys/net/ipv4/icmp_echo_ignore_broadcasts
  for i in /proc/sys/net/ipv4/conf/*/rp_filter; do
        echo "1" > $i
  done
  for i in /proc/sys/net/ipv4/conf/*/log_martians; do
        echo "1" > $i
  done
  for i in /proc/sys/net/ipv4/conf/*/accept_source_route; do
        echo "0" > $i
  done
  for i in /proc/sys/net/ipv4/conf/*/accept_redirects; do
        echo "0" > $i
  done
  for i in /proc/sys/net/ipv4/conf/*/send_redirects; do
        echo "0" > $i
  done

# 2. 清除规则、设定预设政策及开放 lo 与相关的设定值
  PATH=/sbin:/usr/sbin:/bin:/usr/bin; export PATH
  iptables -F
  iptables -X
  iptables -Z
  iptables -P INPUT   DROP
  iptables -P OUTPUT  ACCEPT
  iptables -P FORWARD ACCEPT
  iptables -A INPUT -i lo -j ACCEPT
  iptables -A INPUT -m state --state RELATED -j ACCEPT

# 3. 启动额外的防火墙 script 模块
  if [ -f /usr/local/firewall/iptables.deny ]; then
        sh /usr/local/firewall/iptables.deny
  fi
  if [ -f /usr/local/firewall/iptables.allow ]; then
        sh /usr/local/firewall/iptables.allow
  fi
  if [ -f /usr/local/firewall/iptables.http ]; then
        sh /usr/local/firewall/iptables.http
  fi

  iptables -A INPUT -m state --state ESTABLISHED -j ACCEPT

# 4. 允许某些类型的 ICMP 封包进入
  AICMP="0 3 3/4 8 4 11 12 14 16 18"
  for tyicmp in $AICMP
  do
     iptables -A INPUT -i $EXTIF -p icmp --icmp-type $tyicmp -j ACCEPT
  done

# 5. 允许某些服务的进入，请依照您自己的环境开启

  iptables -A INPUT -p TCP -i $EXTIF --dport 3306 -j ACCEPT   # MYSQL
  iptables -A INPUT -p TCP -i $EXTIF --dport 22   -j ACCEPT   # SSH
  iptables -A INPUT -p TCP -i $EXTIF --dport 80   -j ACCEPT   # WWW
  iptables -A INPUT -p TCP -i $EXTIF --dport 873  -j ACCEPT   # RSYNC

# 防止SYN攻击 轻量
  iptables -N syn-flood
  iptables -A INPUT -p TCP --syn -j syn-flood
  iptables -I syn-flood -p TCP -m limit --limit 3/s --limit-burst 6 -j RETURN
  iptables -A syn-flood -j REJECT

# 为了防止DOS太多连接进来,那么可以允许最多15个初始连接,超过的丢弃
# iptables -A INPUT -s $INNET -p TCP -m state --state ESTABLISHED,RELATED -j ACCEPT
# iptables -A INPUT -i $EXTIF -p TCP --syn -m connlimit --connlimit-above 15 -j DROP
# iptables -A INPUT -s $INNET -p TCP --syn -m connlimit --connlimit-above 15 -j DROP

#设置icmp阔值 ,并对攻击者记录在案
  iptables -A INPUT -p icmp -m limit --limit 3/s -j LOG --log-level INFO --log-prefix "ICMP packet IN:"
  iptables -A INPUT -p icmp -m limit --limit 6/m -j ACCEPT
  iptables -A INPUT -p icmp -j DROP

# 防止端口扫描
# iptables -I INPUT -p icmp --icmp-type echo-request -m state --state NEW -j DROP

#标志为FIN，URG，PSH拒绝
  iptables -A INPUT -i $EXTIF -p TCP --tcp-flags SYN,RST SYN,RST -j DROP
  iptables -A INPUT -i $EXTIF -p TCP --tcp-flags SYN,FIN SYN,FIN -j DROP
  iptables -A INPUT -i $EXTIF -p TCP --tcp-flags ALL ALL -j DROP
  iptables -A INPUT -i $EXTIF -p TCP --tcp-flags ALL SYN,RST,ACK,FIN,URG -j DROP
  iptables -A INPUT -i $EXTIF -p TCP  --tcp-flags ALL NONE -j DROP

# 第二部份，针对后端主机的防火墙设定！##############################
# 1. 先加载一些有用的模块
  modules="ip_tables iptable_nat ip_nat_ftp ip_nat_irc ip_conntrack ip_conntrack_ftp ip_conntrack_irc"
  for mod in $modules
  do
      testmod=`lsmod | grep "${mod} "`
      if [ "$testmod" == "" ]; then
              modprobe $mod
      fi
  done

# 2. 清除 NAT table 的规则吧！
  iptables -F -t nat
  iptables -X -t nat
  iptables -Z -t nat
  iptables -t nat -P PREROUTING  ACCEPT
  iptables -t nat -P POSTROUTING ACCEPT
  iptables -t nat -P OUTPUT      ACCEPT
EOF
```

随机启动 iptables：

```sh
echo "/usr/local/firewall/iptables.rule" >> /etc/rc.local
```

### 6.2. PHP 安全配置

修改 php.ini 配置文件：

    disable_functions = exec,passthru,shell_exec,system,proc_open,popen,curl_exec,curl_multi_exec,parse_ini_file,show_source

php.ini 官方手册参考 [Description of core php.ini directives][phpini]

[phpini]: http://php.net/manual/en/ini.core.php

### 6.3. MySQL 安全配置



### 6.4. CentOS 系统安全配置

**禁止更新 kernel 相关的包**

```sh
echo "exclude=kernel*" >> /etc/yum.conf
```

### 7. 运维笔记


把 PHP 和 MySQL 相关命令添加到 PATH 中：

```sh
export PATH=$PATH:/usr/local/webserver/php/bin:/usr/local/webserver/mysql/bin
echo "export PATH=$PATH:/usr/local/webserver/php/bin:/usr/local/webserver/mysql/bin" >> ~/.bashrc
```







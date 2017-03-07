# How to Install nginx with PHP 7 and MySQL 5.7 on CentOS 7

## 0. 约定

- 暂用系统用户 root 方便安装调试
- 主机IP地址：192.168.187.10
- 软件包下载到 `/usr/src/` 目录下
- 所有软件包安装在 `/usr/local/webserver/` 目录下
- `/u01/mysql/` 存放数据库
- `/u01/www/` 存放网站数据
- `/u01/logfiles/` 存放网站日志

## 1. 系统要求

系统环境：

- CentOS 7.3.1611

所需软件包：

    nginx-1.10.3.tar.gz
    php-7.1.2.tar.gz
    mysql-boost-5.7.17.tar.gz
    redis-3.2.8.tar.gz
    node-v6.10.0.tar.gz
    Python-3.6.0.tgz 

相关库：

    ImageMagick.tar.gz
    imagick-3.4.3.tgz
    libiconv-1.14.tar.gz
    libmcrypt-2.5.8.tar.gz
    mcrypt-2.6.8.tar.gz
    mhash-0.9.9.9.tar.gz
    pcre-8.39.tar.gz
    redis-3.1.1
    libedit-20160903-3.1.tar.gz
    mongodb-1.2.5
    
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
echo "nameserver 233.5.5.5" >> /etc/resolv.conf
echo "nameserver 8.8.8.8" >> /etc/resolv.conf
```

### 2.2. 禁用 SELinux

```sh
setenforce 0
sed -i 's/^SELINUX=.*$/SELINUX=disabled/' /etc/selinux/config
```

### 2.3. 禁用 IPv6
```sh
sysctl -w net.ipv6.conf.all.disable_ipv6=1
sysctl -w net.ipv6.conf.default.disable_ipv6=1
```

### 2.4. 进程打开文件数
```sh
cat >> /etc/security/limits.conf <<EOF
* soft nproc 65536
* hard nproc 65536
* soft nofile 65536
* hard nofile 65536
EOF
```

## 3. 安装前的准备

### 3.1. 编译工具及相关库

```sh
sudo -s
LANG=C
yum -y install gcc gcc-c++ autoconf automake cmake zlib zlib-devel compat-libstdc++-33 glibc glibc-devel glib2 glib2-devel bzip2 bzip2-devel ncurses ncurses-devel curl curl-devel libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel libtiff-devel gd gd-devel libxml2 libxml2-devel libXpm libXpm-devel libmcrypt libmcrypt-devel readline-devel krb5 krb5-devel libidn libidn-devel openssl openssl-devel openldap openldap-devel nss_ldap openldap-clients openldap-servers pam-devel libicu libicu-devel
```

### 3.2. 下载所需软件包

下载：
```sh
cd /usr/src/
wget http://nginx.org/download/nginx-1.10.3.tar.gz
wget -O php-7.1.2.tar.gz http://cn2.php.net/distributions/php-7.1.2.tar.gz
wget http://mirrors.sohu.com/mysql/MySQL-5.7/mysql-boost-5.7.17.tar.gz
wget http://ftp.gnu.org/gnu/libiconv/libiconv-1.14.tar.gz
wget http://downloads.sourceforge.net/mcrypt/libmcrypt-2.5.8.tar.gz
wget http://downloads.sourceforge.net/mcrypt/mcrypt-2.6.8.tar.gz
wget http://downloads.sourceforge.net/mhash/mhash-0.9.9.9.tar.gz
wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.38.tar.gz
wget http://www.imagemagick.org/download/ImageMagick.tar.gz
wget https://pecl.php.net/get/imagick-3.4.3.tgz
wget http://download.redis.io/releases/redis-3.2.8.tar.gz
wget https://pecl.php.net/get/redis-3.1.1.tgz
wget http://thrysoee.dk/editline/libedit-20160903-3.1.tar.gz
wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel70-3.4.2.tgz
wget https://pecl.php.net/get/mongodb-1.2.5.tgz
wget https://nodejs.org/dist/v6.10.0/node-v6.10.0.tar.gz
wget http://mirrors.sohu.com/python/3.6.0/Python-3.6.0.tgz
```

全部解压：
```sh
cd /usr/src/
cat *.gz *.tgz | tar zxf - -i
```
### 3.3. 安装常用工具

安装 Vim 等工具：

```sh
yum -y install man wget unzip zip net-tools nmap iptraf iotop htop sysstat ntp vim-enhanced bash-completion
```

简单配置 Vim：

```sh
alias vi='vim'
echo "alias vi='vim'" >> ~/.bashrc
echo "set fencs=utf-8,ucs-bom,gb18030,gbk,gb2312,cp936" >> ~/.vimrc
echo "set fileformats=unix,dos" >> ~/.vimrc
```

## 4. 安装

### 4.1. 相关库

```sh
cd /usr/src/libiconv-*/
./configure --prefix=/usr/local
make
sed -i '/gets is a security hole/c\#if defined(__GLIBC__) && !defined(__UCLIBC__) && !__GLIBC_PREREQ(2, 16)\n_GL_WARN_ON_USE (gets, "gets is a security hole - use fgets instead");\n#endif' srclib/stdio.h
make && make install

cd /usr/src/libmcrypt-*/
./configure && make && make install
/sbin/ldconfig
cd libltdl/
./configure --enable-ltdl-install
make && make install

ln -s /usr/local/lib/libmcrypt.la /usr/lib/libmcrypt.la
ln -s /usr/local/lib/libmcrypt.so /usr/lib/libmcrypt.so
ln -s /usr/local/lib/libmcrypt.so.4 /usr/lib/libmcrypt.so.4
ln -s /usr/local/lib/libmcrypt.so.4.4.8 /usr/lib/libmcrypt.so.4.4.8
ln -s /usr/local/lib/libmhash.a /usr/lib/libmhash.a
ln -s /usr/local/lib/libmhash.la /usr/lib/libmhash.la
ln -s /usr/local/lib/libmhash.so /usr/lib/libmhash.so
ln -s /usr/local/lib/libmhash.so.2 /usr/lib/libmhash.so.2
ln -s /usr/local/lib/libmhash.so.2.0.1 /usr/lib/libmhash.so.2.0.1

cd /usr/src/mhash-*/
./configure && make && make install

cd /usr/src/mcrypt-*/
/sbin/ldconfig
./configure && make && make install
echo "/usr/local/lib" >> /etc/ld.so.conf
/sbin/ldconfig
```

### 4.2. 安装 MySQL 5.7

正式安装前先建立 `mysql:mysql` 用户：

```sh
/usr/sbin/groupadd mysql
/usr/sbin/useradd -g mysql -s /sbin/nologin mysql
```

解压前配置软件包

```sh
cd /usr/src/mysql-5.7.*/
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
-DWITH_BOOST=boost/boost_1_59_0 \
-DENABLE_DOWNLOADS=1
```

如果主要用于移动端数据存储，可以默认 utf8mb4 编码，可存储 emoji 表情符号：

```sh
cmake \
-DCMAKE_INSTALL_PREFIX=/usr/local/webserver/mysql \
-DMYSQL_DATADIR=/u01/mysql \
-DEXTRA_CHARSETS=all \
-DDEFAULT_CHARSET=utf8mb4 \
-DDEFAULT_COLLATION=utf8mb4_general_ci \
-DMYSQL_TCP_PORT=3306 \
-DMYSQL_USER=mysql \
-DSYSCONFDIR=/etc \
-DWITH_MYISAM_STORAGE_ENGINE=1 \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_PARTITION_STORAGE_ENGINE=1 \
-DWITH_ARCHIVE_STORAGE_ENGINE=1 \
-DWITH_MEMORY_STORAGE_ENGINE=1 \
-DWITH_BOOST=boost/boost_1_59_0 \
-DENABLE_DOWNLOADS=1
```

编译安装：
```sh
make -j `nproc` && make install
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

log_bin = /u01/mysql/binlogs/mysql-bin
binlog_format = mixed
expire_logs_days = 0
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

mkdir -p /u01/mysql/binlogs
chow mysql:mysql /u01/mysql/binlogs
```

将MySQL数据库的动态链接库共享至系统链接库：

```sh
echo "/usr/local/webserver/mysql/lib/" > /etc/ld.so.conf.d/mysql.conf
/sbin/ldconfig
/sbin/ldconfig -v | grep mysql
```

启动 MySQL：

```sh
/etc/init.d/mysqld start
```

给 MySQL root 用户添加密码：

```sh
DBROOTPWD=123456
/usr/local/webserver/mysql/bin/mysql -e "grant all privileges on *.* to root@'127.0.0.1' identified by \"$DBROOTPWD\" with grant option;"
/usr/local/webserver/mysql/bin/mysql -e "grant all privileges on *.* to root@'localhost' identified by \"$DBROOTPWD\" with grant option;"
```

关闭 MySQL：

```sh
/etc/init.d/mysqld stop
```

### 4.3. 安装 PHP7

先建立 www:www 用户：

```sh
/usr/sbin/groupadd www
/usr/sbin/useradd -g www -s /sbin/nologin www
mkdir -p /u01/www
chmod +w /u01/www
chown -R www:www /u01/www
```

安装 PHP：

```sh
cd /usr/src/php-7.*/
./configure --prefix=/usr/local/webserver/php7 \
--with-config-file-path=/usr/local/webserver/php7/etc \
--with-libdir=lib64 \
--enable-fpm \
--with-fpm-user=www \
--with-fpm-group=www \
--enable-mysqlnd \
--with-mysqli=mysqlnd \
--enable-pdo \
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
--with-readline \
--disable-ipv6
```

**注意：**PHP 默认配置文件存放在 `php7/lib/` 下，为什么不放在 `php7/etc/` 下？为什么不放在 `php7/etc/` 下？为什么？好，我把它改到 `php7/etc/` 下，所以上面编译选项中我添加了这一项 `--with-config-file-path=/usr/local/webserver/php7/etc`。如果你觉得默认就很爽，那你大可以把这项去掉，能很欢快地找到 php.ini 并使之生效即可，如果发现配置不生效，请用 `php --ini` 命令 check 一下。

编译安装：

```sh
make ZEND_EXTRA_LIBS='-liconv' -j `nproc` && make install
```

php.ini 和 php-fpm.conf

```sh
cp php.ini-production /usr/local/webserver/php7/etc/php.ini
cp /usr/local/webserver/php7/etc/php-fpm.conf.default /usr/local/webserver/php7/etc/php-fpm.conf
cp /usr/local/webserver/php7/etc/php-fpm.d/www.conf.default /usr/local/webserver/php7/etc/php-fpm.d/www.conf
```

php-fpm.conf 配置 `vi /usr/local/webserver/php7/etc/php-fpm.conf`：

    [global]
    pid = run/php-fpm.pid
    error_log = log/php-fpm.log
    emergency_restart_threshold = 60
    emergency_restart_interval = 60
    process_control_timeout = 5s
    daemonize = yes
    rlimit_files = 65535

www.conf 配置 `vi /usr/local/webserver/php7/etc/php-fpm.d/www.conf`：

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
sed -i '/^;pid/s/^;//' /usr/local/webserver/php7/etc/php-fpm.conf
sed -i '/^;error_log/s/^;//' /usr/local/webserver/php7/etc/php-fpm.conf
sed -i '/^;emergency_restart_threshold/c\emergency_restart_threshold = 60' /usr/local/webserver/php7/etc/php-fpm.conf
sed -i '/^;emergency_restart_interval/c\emergency_restart_interval = 60' /usr/local/webserver/php7/etc/php-fpm.conf
sed -i '/^;process/c\process_control_timeout = 5s' /usr/local/webserver/php7/etc/php-fpm.conf
sed -i '/^;daemonize/s/^;//' /usr/local/webserver/php7/etc/php-fpm.conf
sed -i '/^;rlimit_files/c\rlimit_files = 65535' /usr/local/webserver/php7/etc/php-fpm.conf

sed -i '/^pm =/c\pm = static' /usr/local/webserver/php7/etc/php-fpm.d/www.conf.default
sed -i '/^pm.max_children/s/[0-9]\+$/256/' /usr/local/webserver/php7/etc/php-fpm.d/www.conf.default
sed -i '/^pm.start_servers/s/[0-9]\+$/20/' /usr/local/webserver/php7/etc/php-fpm.d/www.conf.default
sed -i '/^pm.min_spare_servers/s/[0-9]\+$/5/' /usr/local/webserver/php7/etc/php-fpm.d/www.conf.default
sed -i '/^pm.max_spare_servers/s/[0-9]\+$/35/' /usr/local/webserver/php7/etc/php-fpm.d/www.conf.default
sed -i '/^;pm.process_idle_timeout/s/^;//' /usr/local/webserver/php7/etc/php-fpm.d/www.conf.default
sed -i '/^;pm.max_requests/c\pm.max_requests = 51200' /usr/local/webserver/php7/etc/php-fpm.d/www.conf.default
sed -i '/^;request_terminate_timeout/s/^;//' /usr/local/webserver/php7/etc/php-fpm.d/www.conf.default
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
cd /usr/src/ImageMagick-*/
./configure && make && make install
/sbin/ldconfig

cd /usr/src/imagick-*/
/sbin/ldconfig
/usr/local/webserver/php7/bin/phpize
./configure --with-php-config=/usr/local/webserver/php7/bin/php-config
make && make install

cd /usr/src/redis-3.1.0/
/usr/local/webserver/php7/bin/phpize
./configure --with-php-config=/usr/local/webserver/php7/bin/php-config
make && make install

cd /usr/src/mongodb-1.2.2/
/usr/local/webserver/php7/bin/phpize
./configure --with-php-config=/usr/local/webserver/php7/bin/php-config
make && make install
```

编译安装好模块，还要在 `php.ini` 里添加这些模块，使之生效：

```sh
vi /usr/local/webserver/php7/etc/php.ini
```

配置项：

    ; extension_dir = "ext"
    extension_dir = "/usr/local/webserver/php7/lib/php/extensions/no-debug-non-zts-20151012/"
    extension=imagick.so
    extension=redis.so
    extension=mongodb.so

再次注意 `php.ini` 的位置，这个真的很重要！

还是在 `php.ini` 文件中，启用内置的 opcache 模块，并调整相关配置，可以直接在`[opcache]`这一行下添加如下配置项：

    zend_extension=opcache.so
    opcache.enable=1
    opcache.memory_consumption=256
    opcache.interned_strings_buffer=8
    opcache.max_accelerated_files=4000
    opcache.revalidate_freq=60
    opcache.fast_shutdown=1

以上也可以直接用下面两条 sed 命令修改：

```sh
sed -i '/; extension_dir = "ext"/a\extension_dir = "/usr/local/webserver/php7/lib/php/extensions/no-debug-non-zts-20160303/"\nextension=imagick.so\nextension=redis.so\nextension=mongodb.so' /usr/local/webserver/php7/etc/php.ini
sed -i '/\[opcache\]/a\\nzend_extension=opcache.so\nopcache.enable=1\nopcache.memory_consumption=256\nopcache.interned_strings_buffer=8\nopcache.max_accelerated_files=4000\nopcache.revalidate_freq=60\nopcache.fast_shutdown=1\n' /usr/local/webserver/php7/etc/php.ini
```

重新启动 php-fpm 即可生效。

### 4.4. 安装 nginx

先要安装 PCRE 支持：

```sh
cd /usr/src/pcre-*/
./configure && make && make install
```

配置并安装 nginx：

```sh
cd /usr/src/nginx-*/
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
cat > /usr/local/webserver/nginx/conf/nginx.conf <<\EOF
user  www www;

#worker_processes 8;
worker_rlimit_nofile 65535;

error_log  /u01/logfiles/nginx/nginx_error.log  warn;

events
{
  use epoll;
  worker_connections 65535;
}

http
{
  include       mime.types;
  default_type  application/octet-stream;

  server_names_hash_bucket_size 128;
  client_header_buffer_size 32k;
  large_client_header_buffers 4 32k;
  client_max_body_size 8m;

#  limit_conn_zone $binary_remote_addr zone=connlimit:9m;

  server_tokens off;
  sendfile on;
  tcp_nopush on;

  keepalive_timeout 10;

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
    listen 80;
    server_name example.com;
    index index.php index.html;
    root /u01/www/;

    #limit_conn  connlimit 15;
    #limit_rate  256k;

    location ~ .*\.php$
    {
      fastcgi_pass 127.0.0.1:9000;
      fastcgi_index index.php;
      include fastcgi.conf;
    }

    location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$
    {
      expires      30d;
      access_log   off;
    }
    location ~ .*\.(js|css)$
    {
      expires      1h;
      access_log   off;
    }

    access_log  /u01/logfiles/nginx/access.log  access;
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
log_path="/u01/logfiles/nginx"
new_log_path=${log_path}/$(date -d 'yesterday' '+%Y%m%d')

mkdir -p ${new_log_path}
mv ${log_path}/*.log ${new_log_path}

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

修改 `disable = yes` 为 `disable = no`

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


### 5.4. Redis

编译安装：
```sh
cd /usr/src/redis-3.2.8/
make && make install
```

建立 redis data 目录：

```sh
mkdir -p /u01/redis/
```

安装启动脚本（注意其中的 conf 文件、日志文件、数据目录等的位置）：
```
# ./install_server.sh
Welcome to the redis service installer
This script will help you easily set up a running redis server

Please select the redis port for this instance: [6379]
Selecting default: 6379
Please select the redis config file name [/etc/redis/6379.conf]
Selected default - /etc/redis/6379.conf
Please select the redis log file name [/var/log/redis_6379.log] /u01/logfiles/redis.log
Please select the data directory for this instance [/var/lib/redis/6379] /u01/redis
Please select the redis executable path [/usr/local/bin/redis-server]
Selected config:
Port           : 6379
Config file    : /etc/redis/6379.conf
Log file       : /u01/logfiles/redis.log
Data dir       : /u01/redis
Executable     : /usr/local/bin/redis-server
Cli Executable : /usr/local/bin/redis-cli
Is this ok? Then press ENTER to go on or Ctrl-C to abort.
Copied /tmp/6379.conf => /etc/init.d/redis_6379
Installing service...
Successfully added to chkconfig!
Successfully added to runlevels 345!
Starting Redis server...
Installation successful!
```

启动 redis-server：
```sh
chkconfig redis_6379 on
/etc/init.d/redis_6379 start
```

### 5.5. node.js

```sh
cd /usr/src/node-*/
./configure && make && make install
```

### 5.6. 安装 Elasticsearch

Elasticsearch 是 JVM 平台的开源搜索引擎，安装它之前要先安装 Java 环境，下载 `jdk-8u112-linux-x64.tar.gz`，解压至 `/usr/local/jdk1.8.0_112`，配置 JDK 环境：

```sh
cat >> /etc/profile <<'EOF'

export JAVA_HOME=/usr/local/jdk1.8.0_112
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:${PATH}
EOF

source /etc/profile
```

查看是否安装成功：
```sh
java -version
```

下载 Elasticsearch，并解压至 `/usr/local/webserver/es`，新建`elastic`用户，用来启动 Elasticsearch，官方不建议直接使用 `root` 用户启动之：

```sh
useradd elastic
chown -R elastic:elastic /usr/local/webserver/es
su elastic
```

开发机可以根据实际情况更改 JVM 空间分配，这里设为 512MB：
```sh
sed -i 's/-Xms2g/-Xms512m/;s/-Xmx2g/-Xmx512m/' /usr/local/webserver/es/config/jvm.options
```

Elasticsearch 启动前的必要配置：

```sh
ulimit -n 65536
echo "vm.max_map_count=262144" >> /etc/sysctl.conf
sysctl -p
```

启动 Elasticsearch：
```sh
/usr/local/webserver/es/bin/elasticsearch -d
```

### 5.7. 安装 mongodb

安装：
```sh
mv /usr/src/mongodb-linux-x86_64-rhel70-3.4.1 /usr/local/webserver/mongodb

cat > /etc/mongod.conf <<'EOF'
# mongod.conf
dbpath=/u01/mongodb

port = 27017

#where to log
logpath=/u01/logfiles/mongodb.log
logappend = true

#rest = true

verbose = true
## for log , more verbose
##vvvvv = true
#
##profile = 2
##slowms = 10

# fork and run in background
fork = true

# Disables write-ahead journaling
# nojournal = true

# Enables periodic logging of CPU utilization and I/O wait
#cpu = true

# Turn on/off security.  Off is currently the default
#noauth = true
#auth = true

# Verbose logging output.
#verbose = true

# Inspect all client data for validity on receipt (useful for
# developing drivers)
#objcheck = true

# Enable db quota management
#quota = true

# Set oplogging level where n is
#   0=off (default)
#   1=W
#   2=R
#   3=both
#   7=W+some reads
#oplog = 0

# Ignore query hints
#nohints = true

# Disable the HTTP interface (Defaults to localhost:27018).
nohttpinterface = true

# Turns off server-side scripting.  This will result in greatly limited
# functionality
#noscripting = true

# Turns off table scans.  Any query that would do a table scan fails.
#notablescan = true

# Disable data file preallocation.
#noprealloc = true

# Specify .ns file size for new databases.
# nssize = <size>

# Accout token for Mongo monitoring server.
#mms-token = <token>

# Server name for Mongo monitoring server.
#mms-name = <server-name>

# Ping interval for Mongo monitoring server.
#mms-interval = <seconds>

# Replication Options

# in replicated mongo databases, specify here whether this is a slave or master
#slave = true
#source = master.example.com
# Slave only: specify a single database to replicate
#only = master.example.com
# or
#master = true
#source = slave.example.com
EOF

mkdir -p /u01/mongodb/
```

禁用大内存页面：
```sh
echo never > /sys/kernel/mm/transparent_hugepage/enabled
echo never > /sys/kernel/mm/transparent_hugepage/defrag

cat >> /etc/rc.local <<'EOF'

if test -f /sys/kernel/mm/transparent_hugepage/enabled; then
   echo never > /sys/kernel/mm/transparent_hugepage/enabled
fi
if test -f /sys/kernel/mm/transparent_hugepage/defrag; then
   echo never > /sys/kernel/mm/transparent_hugepage/defrag
fi

EOF
```

mongodb 相关命令：
```sh
export PATH=$PATH:/usr/local/webserver/mongodb/bin
echo 'export PATH=$PATH:/usr/local/webserver/mongodb/bin' >> ~/.bashrc
```

随机启动 mongod：
```sh
echo '/usr/local/webserver/mongodb/bin/mongod --config /etc/mongod.conf' >> /etc/rc.local
```

运行：
```sh
/usr/local/webserver/mongodb/bin/mongod --config /etc/mongod.conf
```

关闭：
```sh
pkill mongod
```

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

## 7. 其他工具安装

### 7.1. 安装 node.js

```sh
cd /usr/src/
wget https://nodejs.org/dist/v4.4.7/node-v4.4.7.tar.gz
tar zxvf node-v4.4.7.tar.gz
cd node-v4.4.7/
./configure
make && make install
```

安装 cnpm：

```sh
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

### 7.2. 安装 composer

**注意：请不要在`root`权限下使用 composer，更不要在线上使用 composer：**
```sh
curl -sS https://getcomposer.org/installer | php
mv composer.phar /usr/local/bin/composer
```

### 7.3. 安装 Git

安装 Git 需要 libiconv 支持，上述步骤中已安装过，接下来安装 Git：

```sh
cd /usr/src/git-*/
./configure --prefix=/usr/local --with-iconv=/usr/local/lib
make && make install
```

在安装的时候，可能会因为有没有 Perl 的 ExtUtils::MakeMaker 模块，而出错：

    # make
        SUBDIR perl
    /usr/bin/perl Makefile.PL PREFIX='/usr/local' INSTALL_BASE='' --localedir='/usr/local/share/locale'
    Can't locate ExtUtils/MakeMaker.pm in @INC (@INC contains: /usr/local/lib64/perl5 /usr/local/share/perl5 /usr/lib64/perl5/vendor_perl /usr/share/perl5/vendor_perl /usr/lib64/perl5 /usr/share/perl5 .) at Makefile.PL line 3.
    BEGIN failed--compilation aborted at Makefile.PL line 3.
    make[1]: *** [perl.mak] Error 2
    make: *** [perl/perl.mak] Error 2

yum 安装一下即可：

```sh
yum -y install perl-ExtUtils-MakeMaker
```

### 7.4. 安装 Ruby/Sass/Gulp.js

```sh
yum -y install ruby
gem sources --add https://gems.ruby-china.org/ --remove https://rubygems.org/
gem sources -l
gem install sass
cnpm install -g gulp
```

### 7.5. 安装 Python 3.6.0/pip

```sh
cd /usr/src/Python-3.6.0/
./configure --prefix=/usr/local && make && make altinstall
python3.6 --version
easy_install-3.6 pip
pip3.6 --version
```

## 8. 其他


把 PHP 和 MySQL 相关命令添加到 PATH 中：

```sh
cat >> ~/.bashrc <<'EOF'

export PATH=$PATH:/usr/local/webserver/php7/bin
export PATH=$PATH:/usr/local/webserver/mysql/bin
export PATH=$PATH:/usr/local/webserver/redis/src
EOF
```

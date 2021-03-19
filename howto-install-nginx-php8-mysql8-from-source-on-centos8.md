# LEMP: How to Install nginx with PHP 8 and MySQL 8 from Source on CentOS 8

## 0. 约定

- 暂用系统用户 root 方便安装调试
- 主机IP地址：192.168.111.106
- 软件包下载到 `/usr/src/` 目录下
- 所有软件包安装在 `/opt/` 目录下
- `/data/mysql/` 存放MySQL数据，类似的还有 `/data/redis` 和 `/data/mongodb` 等
- `/data/www/` 存放网站数据
- `/data/log/` 存放网站日志

## 系统基本设定

### 网络设置

网络设置：

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
IPADDR=192.168.111.106
NETMASK=255.255.255.0
GATEWAY=192.168.111.2
DNS1=192.168.111.2

systemctl restart NetworkManager.service
```

### 时区

```sh
timedatectl list-timezones
timedatectl set-timezone Asia/Shanghai
```

### 时间同步服务

```sh
dnf -y install chrony
systemctl enable chronyd.service
systemctl start chronyd.service
```

### 禁用 SELinux

```sh
setenforce 0
sed -i 's/^SELINUX=.*$/SELINUX=disabled/' /etc/selinux/config
```

### 进程打开文件数量限制
```sh
ulimit -n 65535
sed -i '/DefaultLimitNOFILE=/c\DefaultLimitNOFILE=65535' /etc/systemd/system.conf
```

### 禁用防火墙

```sh
systemctl disable firewalld.service
systemctl stop firewalld.service
```

## 安装前的准备

### 安装常用工具

安装常用工具：

```sh
dnf -y install tar wget unzip zip net-tools nmap iptraf iotop sysstat bash-completion
```

安装配置 Vim：

```sh
dnf -y install vim-enhanced
alias vi='vim'
echo "alias vi='vim'" >> ~/.bashrc
echo "set fencs=utf-8,ucs-bom,gb18030,gbk,gb2312,cp936" >> ~/.vimrc
echo "set fileformats=unix,dos" >> ~/.vimrc
```

### 编译工具及相关库

```sh
dnf -y install gcc gcc-c++ autoconf automake make cmake bison zlib zlib-devel glibc glibc-devel glib2 glib2-devel bzip2 bzip2-devel libzip ncurses ncurses-devel curl curl-devel libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel libtiff-devel gd gd-devel libxml2 libxml2-devel libxslt libxslt-devel libXpm libXpm-devel readline-devel openssl openssl-devel openldap openldap-devel pam-devel libicu libicu-devel sqlite-devel gmp-devel oniguruma-devel pcre-devel
```

### 下载所需软件包

下载：
```sh
cd /usr/src/
wget http://nginx.org/download/nginx-1.18.0.tar.gz
wget https://www.php.net/distributions/php-8.0.3.tar.gz
wget https://cdn.mysql.com/Downloads/MySQL-8.0/mysql-boost-8.0.23.tar.gz
wget https://ftp.gnu.org/gnu/libiconv/libiconv-1.16.tar.gz
wget https://ftp.pcre.org/pub/pcre/pcre-8.44.tar.gz
wget https://zlib.net/zlib-1.2.11.tar.gz
```

全部解压：
```sh
cd /usr/src/
cat *.gz *.tgz | tar zxf - -i
```

## 安装

### 安装 MySQL 8

启用 PowerTools 存储库，安装相关依赖包：
```sh
dnf install epel-release
dnf config-manager --set-enabled powertools
dnf -y install rpcgen libtirpc-devel
```

查看PowerTools资源库中可用的软件包列表：
```sh
dnf repo-pkgs powertools list
```

创建 mysql 用户：

```sh
/usr/sbin/groupadd mysql
/usr/sbin/useradd -g mysql -s /sbin/nologin mysql
```

配置 MySQL：
```
cd /usr/src/mysql-8.*/
rm CMakeCache.txt
mkdir build
cd build
cmake .. \
-DCMAKE_INSTALL_PREFIX=/opt/mysql \
-DMYSQL_DATADIR=/data/mysql \
-DSYSCONFDIR=/etc \
-DEXTRA_CHARSETS=all \
-DMYSQL_TCP_PORT=3306 \
-DMYSQL_USER=mysql \
-DSYSCONFDIR=/etc \
-DWITH_MYISAM_STORAGE_ENGINE=1 \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_PARTITION_STORAGE_ENGINE=1 \
-DWITH_ARCHIVE_STORAGE_ENGINE=1 \
-DWITH_MEMORY_STORAGE_ENGINE=1 \
-DWITH_BOOST=../boost/boost_1_73_0
```

编译安装：
```sh
make -j `nproc` && make install
```

启动脚本：
```sh
cp /opt/mysql/support-files/mysql.server /etc/init.d/mysqld
chmod +x /etc/init.d/mysqld
chkconfig --add mysqld
chkconfig mysqld on
```

必要的配置：
```sh
cat > /etc/my.cnf <<\EOF
[client]
port   = 3306
socket = /data/mysql/mysql.sock

[mysqld]
ssl       = 0
server-id = 1

port      = 3306
datadir   = /data/mysql
log-error = /data/mysql/mysqld.log
pid-file  = /data/mysql/mysqld.pid
socket    = /data/mysql/mysql.sock

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

初始化数据库（请保持 `/data/mysql/` 目录为空）：

```sh
mkdir -p /data/mysql
chown mysql:mysql /data/mysql/
/opt/mysql/bin/mysqld --initialize --user=mysql --basedir=/opt/mysql --datadir=/data/mysql
```

初始化数据库会在 `/data/mysql/mysqld.log` 中产生 mysql root 账号的初始密码，请立即运行 `/opt/mysql/bin/mysql_secure_installation` 修改 root 密码，并设定其他安全选项。

建立 binlog 日志存放目录：

```sh
mkdir -p /data/log/binlogs
chown mysql:mysql /data/log/binlogs
```

将 MySQL 数据库的动态链接库共享至系统链接库：

```sh
echo "/opt/mysql/lib/" > /etc/ld.so.conf.d/mysql.conf
/sbin/ldconfig
/sbin/ldconfig -v | grep mysql
```

MySQL 启停：

```sh
/etc/init.d/mysqld start
/etc/init.d/mysqld stop
```

### 开始 PHP 8

#### 安装准备工作

**安装依赖包 libiconv**

```sh
cd /usr/src/libiconv-*/
./configure --prefix=/usr/local/libiconv
make && make install
```

**安装指定版本的 libzip**

PHP 安装时 configure 环节，会报如下错误：

    checking for libzip >= 0.11 libzip != 1.3.1 libzip != 1.7.0... no
    configure: error: Package requirements (libzip >= 0.11 libzip != 1.3.1 libzip != 1.7.0) were not met:

    Package 'libzip', required by 'virtual:world', not found

为解决该问题，需把现有 libzip 缷载，安装指定版本：
```sh
remove libzip libzip-devel

cd /usr/src
wget https://libzip.org/download/libzip-1.7.3.tar.gz
tar xzf libzip-1.7.3.tar.gz
cd libzip-1.7.3
mkdir build
cd build
cmake ..
make && make install
export PKG_CONFIG_PATH="/usr/local/lib64/pkgconfig/"
```

#### 安装 PHP 8

先建立 www:www 用户：

```sh
/usr/sbin/groupadd www
/usr/sbin/useradd -g www -s /sbin/nologin www
mkdir -p /data/www
chmod +w /data/www
chown -R www:www /data/www
```

安装 PHP：

```sh
cd /usr/src/php-8.*/
./configure --prefix=/opt/php8 \
--with-libdir=lib64 \
--with-iconv=/usr/local/libiconv \
--enable-fpm \
--with-fpm-user=www \
--with-fpm-group=www \
--enable-mysqlnd \
--with-mysqli=mysqlnd \
--enable-pdo \
--with-pdo-mysql=mysqlnd \
--enable-mbregex \
--with-freetype \
--with-mhash \
--with-readline \
--enable-bcmath \
--with-bz2 \
--enable-calendar \
--with-curl \
--enable-exif \
--enable-ftp \
--with-gettext \
--with-gmp \
--with-iconv \
--with-ldap \
--enable-gd \
--enable-mbstring \
--enable-session \
--enable-shmop \
--enable-soap \
--enable-sockets \
--enable-sysvmsg \
--enable-sysvsem \
--enable-sysvshm \
--enable-opcache \
--with-openssl \
--enable-intl \
--enable-pcntl \
--with-zlib \
--with-zip \
--enable-xml \
--disable-debug \
--disable-phpdbg
```

编译安装：

```sh
make -j `nproc` && make install
```

`php.ini` 和 `php-fpm.conf`

```sh
cp php.ini-production /opt/php8/lib/php.ini
cp /opt/php8/etc/php-fpm.conf.default /opt/php8/etc/php-fpm.conf
cp /opt/php8/etc/php-fpm.d/www.conf.default /opt/php8/etc/php-fpm.d/www.conf
```

`php-fpm.conf` 配置 `vi /opt/php8/etc/php-fpm.conf`：

    [global]
    pid = run/php-fpm.pid
    error_log = log/php-fpm.log
    emergency_restart_threshold = 60
    emergency_restart_interval = 60
    process_control_timeout = 5s
    daemonize = yes
    rlimit_files = 65535

`www.conf` 配置 `vi /opt/php8/etc/php-fpm.d/www.conf`：

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
sed -i \
-e '/^;pid/s/^;//' \
-e '/^;error_log/s/^;//' \
-e '/^;emergency_restart_threshold/c\emergency_restart_threshold = 60' \
-e '/^;emergency_restart_interval/c\emergency_restart_interval = 60' \
-e '/^;process/c\process_control_timeout = 5s' \
-e '/^;daemonize/s/^;//' \
-e '/^;rlimit_files/c\rlimit_files = 65535' \
/opt/php8/etc/php-fpm.conf

sed -i \
-e '/^pm =/c\pm = static' \
-e '/^pm.max_children/s/[0-9]\+$/128/' \
-e '/^pm.start_servers/s/[0-9]\+$/20/' \
-e '/^pm.min_spare_servers/s/[0-9]\+$/5/' \
-e '/^pm.max_spare_servers/s/[0-9]\+$/35/' \
-e '/^;pm.process_idle_timeout/s/^;//' \
-e '/^;pm.max_requests/c\pm.max_requests = 51200' \
-e '/^;request_terminate_timeout/s/^;//' \
/opt/php8/etc/php-fpm.d/www.conf
```

php-fpm 启动脚本：

```sh
cp sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm
chmod +x /etc/init.d/php-fpm
chkconfig --add php-fpm
chkconfig php-fpm on
```

编译安装好模块，还要在 `php.ini` 里添加这些模块，使之生效：

```sh
vi /opt/php8/lib/php.ini
```

配置项：

    ; extension_dir = "ext"
    extension_dir = "/opt/php8/lib/php/extensions/no-debug-non-zts-20200930/"

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
sed -i '/extension_dir = "ext"/a\extension_dir = "/opt/php8/lib/php/extensions/no-debug-non-zts-20200930/"' /opt/php8/lib/php.ini
sed -i '/\[opcache\]/a\\nzend_extension=opcache.so\nopcache.enable=1\nopcache.memory_consumption=256\nopcache.interned_strings_buffer=8\nopcache.max_accelerated_files=4000\nopcache.revalidate_freq=60\nopcache.fast_shutdown=1\n' /opt/php8/lib/php.ini
```

#### PHP 安全配置

修改 php.ini 配置文件：

```sh
sed -i \
-e '/^disable_functions/c\disable_functions = exec,passthru,shell_exec,system,proc_open,popen,curl_exec,curl_multi_exec,parse_ini_file,show_source' \
-e '/;cgi.fix_pathinfo=1/c\cgi.fix_pathinfo=0' \
/opt/php8/lib/php.ini
```

重新启动 php-fpm 即可生效。

### 安装 nginx

先要安装 PCRE 支持：

```sh
cd /usr/src/pcre-*/
./configure && make && make install
```

安装 zlib 依赖包：

```sh
cd /usr/src/zlib-*/
./configure && make && make install
```

配置并安装 nginx：

```sh
cd /usr/src/nginx-*/
./configure \
--prefix=/opt/nginx \
--user=www \
--group=www \
--with-http_ssl_module \
--with-pcre=../pcre-8.44 \
--with-zlib=../zlib-1.2.11
make && make install
```

nginx 配置：

```sh
mkdir -p /data/log/nginx
chmod +w /data/log/nginx
chown -R www:www /data/log/nginx

cp /opt/nginx/conf/nginx.conf{,.original}
cat > /opt/nginx/conf/nginx.conf <<\EOF
user  www www;

#worker_processes 8;
worker_rlimit_nofile 65535;

error_log  /data/log/nginx/nginx_error.log  warn;

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
    root /data/www/;

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

    access_log  /data/log/nginx/access.log  access;
  }

}
EOF
```

nginx 日志轮询：

- nginx 日志存放位置：`/data/log/nginx`
- 日志轮询脚本：`/opt/nginx/cut_nginx_log.sh`
- nginx pid 文件位置：`/opt/nginx/logs/nginx.pid`

```sh
cat > /opt/nginx/cut_nginx_log.sh <<\EOF
#!/bin/bash
# cut_nginx_log.sh
# This script run at 00:00

# The Nginx logs path
log_path="/data/log/nginx"
new_log_path=${log_path}/$(date -d 'yesterday' '+%Y%m%d')

mkdir -p ${new_log_path}
mv ${log_path}/*.log ${new_log_path}

kill -USR1 `cat /opt/nginx/logs/nginx.pid`
EOF
```

crontab 中添加定时任务，00:00 执行日志轮询：

```sh
crontab -e
0 0 * * * /bin/bash /opt/nginx/cut_nginx_log.sh
```

## 其他

### HMTL 和 PHP 测试页面

```sh
cat > /data/www/test.html <<'EOF'
<h1>It works.</h1>
EOF

cat > /data/www/phpinfo.php <<'EOF'
<?php phpinfo();
EOF

chown www:www /data/www/*
```

### 环境变量

把 PHP 和 MySQL 相关命令添加到 PATH 中：

```sh
cat >> ~/.bashrc <<'EOF'

export PATH=$PATH:/opt/php8/bin
export PATH=$PATH:/opt/mysql/bin
export PATH=$PATH:/opt/nginx/sbin
EOF
```

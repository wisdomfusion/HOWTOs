# How to Install Postfix on CentOS 6

## 0. 说明

``好记性不如烂笔头'' 这话什么时候也不过时。几年前，因网站发送验证邮件的需要，折腾了两三天仓促地搭建了 postfix 邮件系统，没头没脑，也没写文档。但回想起来也的确搞笑，本身搭建工作没有什么难度，只不过涉事组件比较多，组件之间还有权限和库文件的依赖，如果能一步步把文档整理出来，事后再遇此事，便再不会慌里慌张。

好了，不再寒暄了。这个文档是真实案例的产物，我在安装过程中也参考了包括 extmail 官方提供的 wiki 文档在内的多篇记笔文档，也就是说，这个文档主要是站在他人肩上做的一套实践活动，随便写了个活动感想。:grin:

## 1. 系统要求

1.1. 系统环境

- CentOS 6.8

1.2. 所需软件包

- httpd
- MySQL 5.7.12
- postfix 3.1.1
- courier-authlib-0.66.4
- dovecot
- cyrus-sasl
- maildrop
- extmail-1.2
- extman-1.1

## 2. 安装前的准备 ##

### 2.1. 编译工具及相关库

```sh
sudo -s
LANG=C
yum -y install gcc gcc-c++ autoconf automake cmake zlib zlib-devel compat-libstdc++-33 glibc glibc-devel glib2 glib2-devel bzip2 bzip2-devel unzip zip nmap ncurses ncurses-devel sysstat ntp curl curl-devel libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel libtiff-devel gd gd-devel libxml2 libxml2-devel libXpm libXpm-devel libmcrypt libmcrypt-devel krb5 krb5-devel libidn libidn-devel openssl openssl-devel openldap openldap-devel nss_ldap openldap-clients openldap-servers pam-devel libicu libicu-devel
yum -y install tcl tcl-devel libart_lgpl libart_lgpl-devel libtool-ltdl libtool-ltdl-devel expect db4-devel
```

### 2.2. 下载一些软件包

```sh
cd /usr/src
wget http://cdn.mysql.com//Downloads/MySQL-5.7/mysql-5.7.12.tar.gz
wget http://downloads.sourceforge.net/project/boost/boost/1.59.0/boost_1_59_0.tar.gz
wget http://downloads.sourceforge.net/project/courier/authlib/0.66.4/courier-authlib-0.66.4.tar.bz2
wget http://cdn.postfix.johnriley.me/mirrors/postfix-release/official/postfix-3.1.1.tar.gz
```

### 2.3. 其他软件包

- extmail-1.2
- extman-1.1

## 3. 安装

### 3.1. 安装 MySQL 5.7

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
tar zxvf mysql-5.7.12.tar.gz
cd mysql-5.7.12/
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

编译安装：

```sh
make -j `grep processor /proc/cpuinfo | wc -l`
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

[mysqld]
port = 3306
socket = /tmp/mysql.sock
basedir = /usr/local/webserver/mysql
datadir = /u01/mysql
pid-file = /u01/mysql/mysql.pid
user = mysql
bind-address = 0.0.0.0
server-id = 1
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

### 3.2. 卸载 sendmail 或 postfix

CentOS 系统自带 sendmail 或 postfix，需要事件卸载，本文采用编译安装方式，软件包安装位置和软件包之间的库依赖都可以自己把控。

```sh
yum remove sendmail
```

或

```sh
yum remove postfix
```

启动 saslauthd服务，并加入自动启动

```sh
chkconfig saslauthd on
/etc/init.d/saslauthd start
```


### 3.3. 安装 postfix


事先添加两个用户和用户组 postfix:postfix 和 postdrop:postdrop，由于 CentOS 6.8 中默认安装过 postfix，所以直接把 postfix 用户和用户组的 id 改成 2525，改成这个数是为了后面配置方便，有些配置直接用的是 uid 和 gid：

添加 `postfix:postfix`
```sh
groupadd -g 2525 postfix
useradd -g postfix -M -s /sbin/nologin -u 2525 postfix
```

或修改 `postfix:postfix`

```sh
usermod -u 2525 postfix
groupmod -g 2525 postfix
```

添加 `postdrop:postdrop`

```sh
groupadd -g 2526 postdrop
useradd -g postdrop -M -s /sbin/nologin -u 2526 postdrop
```

安装 cyrus-sasl

```sh
yum -y install cyrus-sasl
```

```sh
rpm -qa | grep cyrus-sasl
cyrus-sasl-lib-2.1.23-15.el6_6.2.x86_64
cyrus-sasl-2.1.23-15.el6_6.2.x86_64
cyrus-sasl-devel-2.1.23-15.el6_6.2.x86_64
```

```sh
echo "/usr/local/webserver/mysql/lib" >> /etc/ld.so.conf
/sbin/ldconfig
```

```sh
make makefiles 'CCARGS=-DHAS_MYSQL -I/usr/local/webserver/mysql/include -DUSE_SASL_AUTH -DUSE_CYRUS_SASL -I/usr/include/sasl -DUSE_TLS -I/usr/include/openssl' 'AUXLIBS=-L/usr/local/webserver/mysql/lib -lmysqlclient -lz -lm -L/usr/lib64/sasl2 -lsasl2 -lssl -lcrypto'
```

make install

    install_root: [/]
    tempdir: [/usr/src/postfix-3.1.1] /var/tmp/postfix
    config_directory: [/etc/postfix]
    command_directory: [/usr/sbin]
    daemon_directory: [/usr/libexec/postfix]
    data_directory: [/var/lib/postfix]
    html_directory: [no]
    mail_owner: [postfix]
    mailq_path: [/usr/bin/mailq]
    manpage_directory: [/usr/local/man]
    newaliases_path: [/usr/bin/newaliases]
    queue_directory: [/var/spool/postfix]
    readme_directory: [no]
    sendmail_path: [/usr/sbin/sendmail]
    setgid_group: [postdrop]
    shlib_directory: [no]
    meta_directory: [/etc/postfix]


```sh
cat > /etc/init.d/postfix <<\EOF
#!/bin/bash
# postfix script for the postfix server
# chkconfig: 2345 80 30
# description: postfix is the smtp server
. /etc/rc.d/init.d/functions
. /etc/sysconfig/network
[ $NETWORKING = "no" ] && exit 3
[ -x /usr/sbin/postfix ] || exit 4
[ -d /etc/postfix ] || exit 5
[ -d /var/spool/postfix ] || exit 6

RETVAL=0
prog="postfix"

start() {
  echo -n "Starting postfix: "
  /usr/bin/newaliased > /dev/null 2>&1
  /usr/sbin/postfix start 2>/dev/null 1>&2 && success || failure "$prog start"
  #RETVAL=$?
  #[ $RETVAl -eq 0 ] && touch /var/lock/subsys/postfix
  RETVAL=$?
  [ $RETVAL -eq 0 ] && touch /var/lock/subsys/postfix
  echo
  return $RETVAL
}
stop() {
  echo -n "Shutting down postfix: "
  /usr/sbin/postfix stop 2>/dev/null 1>&2 && success || failure "$prog stop"
  RETVAL=$?
  [ $RETVAL -eq 0 ] && rm -f /var/lock/subsys/postfix
  echo
  return $RETVAL
}
reload() {
  echo -n "Reloading postfix: "
  /usr/sbin/postfix reload 2>/dev/null 1>&2 && success || failure "$prog reload"
  RETVAL=$?
  echo
  return $RETVAL
}
abort() {
  /usr/sbin/postfix abort 2>/dev/null 1>&2 && success || failure "$prog abort"
  RETVAL=$?
  echo
  return $RETVAL
}
flush() {
  /usr/sbin/postfix flush 2>/dev/null 1>&2 && success || failure "$prog flush"
  return $RETVAL
}
check() {
  echo -n "Checking postfix"
  /usr/sbin/postfix check 2>/dev/null 1>&2 && success || failure
  RETVAL=$?
  echo
  return $RETVAL
}
status() {
  /usr/sbin/postfix status
  return $RETVAL

}

restart() {
  stop
  start
}

case "$1" in
  start)
  start
  ;;
  stop)
  stop
  ;;
  restart)
  restart
  ;;
  reload)
  reload
  ;;
  abort)
  abort
  ;;
  flush)
  flush
  ;;
  check)
  check
  ;;
  status)
  status
  ;;
  condrestart)
  [ -f /var/lock/subsys/postfix ] && restart || :
  ;;
  *)
  echo "Usag: $0 {start|stop|restart|reload|abort|flush|check|status|condrestart}"
  exit 2
esac
exit $?
EOF

chmod +x /etc/init.d/postfix
chkconfig --add postfix
chkconfig postfix on
```

### 3.4. 认证


### 3.5. courier-authlib

### 3.6. postfix 支持虚拟域和虚拟用户


### 3.7. dovecot服务



### 3.8. extmail & extman



### 3.9. maildrop


## 4. 




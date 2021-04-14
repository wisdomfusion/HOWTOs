# How to Install nginx with PHP 8.0 and MySQL 8.0 on FreeBSD 13.0

## Basic Settings

```sh
pkg update
pkg upgrade
pkg install -y bash wget curl vim-console git
```

## Install Nginx

```sh
pkg install -y nginx
```
## Install PHP 8.0

```sh
pkg install -y php80 php80-bcmath php80-ctype php80-gd php80-mbstring php80-mysqli php80-opcache php80-openssl php80-pdo php80-tokenizer php80-xml php80-pecl-imagick php80-zip
```

## Install MySQL 8.0

```sh
pkg install -y mysql80-server
mysql_secure_installation
```

## Integrate and Test nginx, PHP and MySQL

Conifg PHP and php-fpm:

```sh
cp /usr/local/etc/php-fpm.d/www.conf{,_bak}
cp /usr/local/etc/php.ini-development /usr/local/etc/php.ini
```

Config nginx:

```sh
```

Config MySQL:

```sh
```


Enable and start service:

```sh
sysrc nginx_enable=yes
sysrc php_fpm_enable=yes
sysrc mysql_enable=yes
service nginx start
service php-fpm start
service mysql-server start
```

Verifying service via network sockets status checking:

```sh
netstat -an -p tcp
```

## Install Node.js

```sh
pkg install -y node14
```

## Install Redis

```sh
pkg install -y redis
sysrc redis_enable=yes
service redis start
```

## Install MongoDB

```sh
pkg install -y mongodb44
sysrc mongod_enable=yes
service mongod start
```

## Install JDK

```sh
pkg install -y openjdk11 openjdk11-jre
```

    ...
    
    This OpenJDK implementation requires fdescfs(5) mounted on /dev/fd and
    procfs(5) mounted on /proc.

    If you have not done it yet, please do the following:

            mount -t fdescfs fdesc /dev/fd
            mount -t procfs proc /proc

    To make it permanent, you need the following lines in /etc/fstab:

            fdesc   /dev/fd         fdescfs         rw      0       0
            proc    /proc           procfs          rw      0       0
    =====
    Message from openjdk11-jre-11.0.10+9.1_1:

    --
    This OpenJDK implementation requires fdescfs(5) mounted on /dev/fd and
    procfs(5) mounted on /proc.

    If you have not done it yet, please do the following:

            mount -t fdescfs fdesc /dev/fd
            mount -t procfs proc /proc

    To make it permanent, you need the following lines in /etc/fstab:

            fdesc   /dev/fd         fdescfs         rw      0       0
            proc    /proc           procfs          rw      0       0


```sh
mount -t fdescfs fdesc /dev/fd
mount -t procfs proc /proc
```

Append to `/etc/fstab`:

    fdesc	/dev/fd		fdescfs		rw	0	0
    proc	/proc		procfs		rw	0	0


# How to Install nginx with PHP 7 and MySQL 5.7 on FreeBSD 11.0

## Basic Settings

```sh
freebsd-update fetch install
pkg install wget curl vim-console git
```

## Install Nginx

```sh
pkg install nginx
sysrc nginx_enable="yes"
service nginx start
```

Verify nginx service network sockets:

```sh
sockstat -4 | grep nginx
```

## Install PHP 7.1

```sh
pkg install php71 php71-bcmath php71-ctype php71-gd php71-json php71-mbstring php71-mcrypt php71-mysqli php71-opcache php71-openssl php71-pdo php71-tokenizer php71-xml php71-xmlrpc php71-pecl-imagick
cp /usr/local/etc/php-fpm.d/www.conf{,_bak}
cp /usr/local/etc/php.ini-development /usr/local/etc/php.ini
```

Run rehash command in order to regenerate the system's cache information about our installed executable files
```sh
rehash
```

```sh
sysrc php_fpm_enable=yes
service php-fpm start
```

Verify php-fpm network sockets:

```sh
sockstat -4 -6 | grep php-fpm
```

## Install MySQL 5.7

```sh
pkg install mysql57-server
sysrc mysql_enable="YES"
service mysql-server start
```

## Integrate and Test nginx, PHP, and MySQL

```sh
netstat -an -p tcp
```

## Sharing Folders on VMware Workstation 15 Pro Between Windows 10 Host and FreeBSD 11.2 Guest

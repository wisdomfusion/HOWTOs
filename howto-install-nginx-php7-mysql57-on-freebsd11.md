# How to Install nginx with PHP 7 and MySQL 5.7 on FreeBSD 11.2

## Basic Settings

```sh
pkg update
pkg upgrade
pkg install --yes bash wget curl vim-console git
```

## Install Nginx

```sh
pkg install --yes nginx
```
## Install PHP 7.1

```sh
pkg install --yes php71 php71-bcmath php71-ctype php71-gd php71-json php71-mbstring php71-mcrypt php71-mysqli php71-opcache php71-openssl php71-pdo php71-tokenizer php71-xml php71-xmlrpc php71-pecl-imagick php71-zip
```

## Install MySQL 5.7

```sh
pkg install --yes mysql57-server
```

## Integrate and Test nginx, PHP, and MySQL

Conifg PHP:

```sh
cp /usr/local/etc/php-fpm.d/www.conf{,_bak}
cp /usr/local/etc/php.ini-development /usr/local/etc/php.ini
```

Run rehash command in order to regenerate the system's cache information about our installed executable files
```sh
rehash
```

Config Nginx:

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

Verifying network sockets status:

```sh
netstat -an -p tcp
```

## Install Node.js


## Install Redis


## Install MongoDB


## Install ELK Stack


## Install Samba Server to Sharing Files with Windows Client

Install Samba server:

```sh
pkg install --yes samba48
```

Create data dir:

```sh
mkdir -p /data/www
chown www:www /data/www
```

Samba server config:

```sh
cat /usr/local/etc/smb4.conf
[global]
workgroup = WORKGROUP
server string = Samba Server Version %v
netbios name = freebsd_shared
wins support = Yes
security = user
hosts allow = 192.168.10.
passdb backend = tdbsam

[www]
path = /data/www
valid users = www
writable  = yes
browsable = yes
read only = no
guest ok = no
public = no
create mask = 0644
directory mask = 0755
```

Enable and start Samba service:

```sh
sysrc samba_server_enable=yes
service samba_server start
```

Use `pdbedit` map system user to Samba user:

```sh
pdbedit -a -u www
```

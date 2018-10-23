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

## Install Redis



## Sharing Folders on VMware Workstation 15 Pro Between Windows 10 Host and FreeBSD 11.2 Guest

```sh
pkg install --yes open-vm-tools-nox11

sysrc vmware_guest_vmblock_enable=yes
sysrc vmware_guest_vmhgfs_enable=yes
sysrc vmware_guest_vmmemctl_enable=yes
sysrc vmware_guest_vmxnet_enable=yes
sysrc vmware_guestd_enable=yes

mkdir /data/shared
vmhgfs-fuse .host:/shared /data/shared -o uid=`id -u www`,gid=`id -g www`,umask=0022,subtype=vmhgfs,allow_other
```

Description of the four kernel modules:

- vmemctl is driver for memory ballooning
- vmxnet is paravirtualized network driver
- vmhgfs is the driver that allows the shared files feature of VMware Workstation and other products that use it.
- vmblock is block filesystem driver to provide drag-and-drop functionality from the remote console.
- VMware Guest Daemon (guestd) is the daemon for controlling communication between the guest and the host including time synchronization.

vmmemctl.ko kernel driver is not available for FreeBSD 11.x in the VMware Tools distributed by VMware.

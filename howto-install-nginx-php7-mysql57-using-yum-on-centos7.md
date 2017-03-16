# How to Install nginx with PHP 7 and MySQL 5.7 Using Yum on CentOS 7

## 网卡名改为 `eth0`

```sh
sed -i '/GRUB_CMDLINE_LINUX/s/"$/ net.ifnames=0 biosdevname=0/' /etc/default/grub
/usr/sbin/grub2-mkconfig -o /boot/grub2/grub.cfg

cat /etc/sysconfig/network-scripts/ifcfg-eth0
TYPE=Ethernet
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=eth0
UUID=e0a41059-6493-4791-8df5-06806c9146c2
DEVICE=eth0
ONBOOT=yes
PEERDNS=yes
PEERROUTES=yes
IPV6_PEERDNS=yes
IPV6_PEERROUTES=yes
IPV6_PRIVACY=no

IPADDR=192.168.187.10
GATEWAY=192.168.187.2
NETMASK=255.255.255.0
DNS1=8.8.8.8
DNS2=114.114.114.114
```

## 禁用 SELinux

```sh
setenforce 0
sed -i 's/^SELINUX=.*$/SELINUX=disabled/' /etc/selinux/config
```

## 禁用 IPv6

```sh
sysctl -w net.ipv6.conf.all.disable_ipv6=1
sysctl -w net.ipv6.conf.default.disable_ipv6=1
```

## 进程打开文件数

```sh
cat >> /etc/security/limits.conf <<EOF
* soft nproc 65536
* hard nproc 65536
* soft nofile 65536
* hard nofile 65536
EOF
```

## 安装 Vim 等工具

```sh
yum -y install man wget unzip zip net-tools nmap iptraf iotop htop sysstat ntp vim-enhanced bash-completion lrzsz
```

**简单配置 Vim**

```sh
alias vi='vim'
echo "alias vi='vim'" >> ~/.bashrc
echo "set fencs=utf-8,ucs-bom,gb18030,gbk,gb2312,cp936" >> ~/.vimrc
echo "set fileformats=unix,dos" >> ~/.vimrc
```

## 安装 epel 源

```sh
yum -y install epel-release
```

## 安装 nginx

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

## 安装 PHP 7

使用 webtatic 源：https://webtatic.com/

```sh
rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
yum -y install php71w-common php71w-cli php71w-fpm php71w-gd php71w-mbstring php71w-mcrypt php71w-mysqlnd php71w-opcache php71w-pdo php71w-xml
```

yum -y install redis

## 安装 MySQL（Percona 分支

```sh
yum install http://www.percona.com/downloads/percona-release/redhat/0.1-4/percona-release-0.1-4.noarch.rpm
yum install Percona-Server-server-57
```


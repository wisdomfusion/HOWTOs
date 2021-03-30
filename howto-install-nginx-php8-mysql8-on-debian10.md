# How to Install nginx with PHP 8 and MySQL 8 (LEMP stack) on Debian 10.9

## Install Debian basic system


```sh
# cat /etc/network/interfaces
```

    # This file describes the network interfaces available on your system
    # and how to activate them. For more information, see interfaces(5).

    source /etc/network/interfaces.d/*

    # The loopback network interface
    auto lo
    iface lo inet loopback

    # The primary network interface
    #allow-hotplug ens33
    #iface ens33 inet dhcp

    auto ens33
    iface ens33 inet static
      address 192.168.111.108
      netmask 255.255.255.0
      gateway 192.168.111.2
      dns-nameservers 192.168.111.2

```sh
# systemctl restart networking.service
```

```sh
# ip -c addr show ens33
```

    2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
        link/ether 00:0c:29:d0:1c:4e brd ff:ff:ff:ff:ff:ff
        inet 192.168.111.108/24 brd 192.168.111.255 scope global ens33
        valid_lft forever preferred_lft forever
        inet 192.168.111.202/24 brd 192.168.111.255 scope global secondary dynamic ens33
        valid_lft 1332sec preferred_lft 1332sec
        inet6 fe80::20c:29ff:fed0:1c4e/64 scope link
        valid_lft forever preferred_lft forever


```sh
$ sudo apt update
```

    [sudo] password for huzhifei:
    huzhifei is not in the sudoers file.  This incident will be reported.


# apt install openssh-server
# apt install sudo
# usermod -aG sudo huzhifei

```sh
$ ssh huzhifei@192.168.111.108
```

After this, sudo still won't work! You will need to logout from that user, then relogin, and sudo will work.

```sh
$ sudo apt update
```

    [sudo] password for huzhifei:
    Hit:1 http://mirrors.ustc.edu.cn/debian buster InRelease
    Hit:2 http://mirrors.ustc.edu.cn/debian buster-updates InRelease
    Hit:3 http://security.debian.org/debian-security buster/updates InRelease
    Reading package lists... Done
    Building dependency tree
    Reading state information... Done
    All packages are up to date.

### tools

$ sudo apt install -y net-tools wget curl vim


## Basic Settings

### ufw

$ sudo apt install ufw

$ sudo ufw default deny incoming
$ sudo ufw default allow outgoing

    Default incoming policy changed to 'deny'
    (be sure to update your rules accordingly)


$ sudo ufw allow ssh

$ sudo ufw enable

    Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
    Firewall is active and enabled on system startup

$ sudo ufw status

    Status: active

    To                         Action      From
    --                         ------      ----
    22/tcp                     ALLOW       Anywhere
    22/tcp (v6)                ALLOW       Anywhere (v6)

##  Install nginx web server


$ sudo apt update
$ sudo apt install -y nginx

sudo ufw allow 'Nginx HTTP'

sudo ufw status

    Status: active

    To                         Action      From
    --                         ------      ----
    22/tcp                     ALLOW       Anywhere
    Nginx HTTP                 ALLOW       Anywhere
    22/tcp (v6)                ALLOW       Anywhere (v6)
    Nginx HTTP (v6)            ALLOW       Anywhere (v6)


## Install mysql-server

Adding the MySQL Software Repository

[MySQL APT Repository](https://dev.mysql.com/downloads/repo/apt/)

mysql-apt-config_0.8.16-1_all.deb

$ cd /tmp
$ wget https://repo.mysql.com//mysql-apt-config_0.8.16-1_all.deb
$ sudo apt install -y lsb-release gnupg
$ sudo dpkg -i mysql-apt-config*.deb
$ sudo apt update



$ sudo apt install -y mysql-server

$ sudo systemctl status mysql

$ mysql_secure_installation

## Install PHP 8

$ sudo apt update
$ sudo apt install -y lsb-release ca-certificates apt-transport-https software-properties-common
$ echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/sury-php.list
$ wget -qO - https://packages.sury.org/php/apt.gpg | sudo apt-key add -

$ sudo apt update
$ sudo apt install -y php8.0

$ sudo systemctl stop apache2.service
$ sudo systemctl disable apache2.service

$ sudo apt install -y php8.0-{mysql,cli,common,mcrypt,bcmath,xml,fpm,curl,mbstring,gd,zip}

$ curl -sS https://getcomposer.org/installer | sudo  php -- --install-dir=/usr/local/bin --filename=composer
$ sudo chmod +x /usr/local/bin/composer

$ cd /var/www
$ sudo composer create-project laravel/laravel debian.wf.com
$ sudo chown -R www-data:www-data /var/www/*

$ cat /etc/nginx/sites-available/default

    server {
        listen 80;
        server_name debian.wf.com;
        root /var/www/debian.wf.com/public;

        index index.php;

        location / {
            try_files $uri $uri/ /index.php?$query_string;
        }

        location ~ \.php$ {
            include snippets/fastcgi-php.conf;
            fastcgi_pass unix:/run/php/php8.0-fpm.sock;
        }
    }

sudo systemctl restart php8.0-fpm.service
sudo systemctl restart nginx.service


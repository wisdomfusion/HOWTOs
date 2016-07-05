# How to Install httpd 2.2 with PHP 5.6 and MySQL 5.7 on Mac OS X

## 0. 简介


## 1. 约定


## 2. 系统要求


## 3. 安装前的准备



## 4. 安装


### 4.1. httpd 2.2

sudo port install apache2

sudo vi /opt/local/apache2/conf/httpd.conf

    ServerName localhost:80
    ...
    DocumentRoot "/Users/huzhifei/Sites"
    ...
    <Directory "/Users/huzhifei/Sites">

sudo vi /opt/local/apache2/conf/extra/httpd-vhosts.conf

    <VirtualHost *:80>
        DocumentRoot "/Users/huzhifei/Sites"
        ServerName localhost
        ErrorLog "logs/localhost-error_log"
        CustomLog "logs/localhost-access_log" common
    </VirtualHost>

```sh
/opt/local/apache2/bin/apachectl -t
sudo port unload apache2
sudo port load apache2
```

### 4.2. MySQL 5.6

```sh
sudo port install mysql56-server
sudo port select mysql mysql56
export PATH=$PATH:/opt/local/lib/mysql56/bin
```

```sh
sudo -u _mysql mysql_install_db
sudo chown -R _mysql:_mysql /opt/local/var/db/mysql56/
sudo chown -R _mysql:_mysql /opt/local/var/run/mysql56/
sudo chown -R _mysql:_mysql /opt/local/var/log/mysql56/
```

```sh
sudo port load mysql56-server
```

```sh
/opt/local/lib/mysql56/bin/mysqladmin -u root -p password
mysql -u root -p

man mysql_secure_installation
/opt/local/bin/mysql_secure_installation
```

database upgrade as necessary:

```sh
sudo port unload mysql56-server
sudo /opt/local/lib/mysql56/bin/mysql_upgrade -u root -p
sudo port load mysql56-server
```

### 4.3. PHP 7.0

```sh
sudo port install php70 php70-apache2handler
sudo port install php70-cgi php70-gd php70-curl php70-iconv php70-gettext php70-mbstring php70-mcrypt php70-mysql php70-openssl php70-sockets php70-zip php70-opcache php70-sqlite
cd /opt/local/etc/php70
sudo cp php.ini-development php.ini
```

sudo vi /opt/local/apache2/conf/httpd.conf

    <IfModule mime_module>
    ...
    AddType application/x-httpd-php .php
    AddType application/x-httpd-php-source .phps
    </IfModule>

```sh
cd /opt/local/apache2/modules
sudo /opt/local/apache2/bin/apxs -a -e -n php7 mod_php70.so
```

sudo vi /opt/local/apache2/conf/httpd.conf

    DirectoryIndex index.php index.html

    # Include PHP configurations
    Include conf/extra/mod_php70.conf

    $ sudo -i
    # cd /opt/local/etc/php70
    # cp php.ini php.ini.bak
    # defSock=$(/opt/local/bin/mysql_config --socket)
    # cat php.ini | sed \
      -e "s#pdo_mysql\.default_socket.*#pdo_mysql\.default_socket=${defSock}#" \
      -e "s#mysql\.default_socket.*#mysql\.default_socket=${defSock}#" \
      -e "s#mysqli\.default_socket.*#mysqli\.default_socket=${defSock}#" > tmp.ini
    # grep default_socket tmp.ini  # Check it!
    default_socket_timeout = 60
    pdo_mysql.default_socket=/opt/local/var/run/mysql56/mysqld.sock
    mysqli.default_socket=/opt/local/var/run/mysql56/mysqld.sock
    # mv tmp.ini php.ini
    # exit # OR rm php.ini.bak && exit


```sh
sudo port unload apache2
sudo port load apache2
export PATH=$PATH:/opt/local/bin
echo 'export PATH=$PATH:/opt/local/bin' ~/.profile
```

port select --list php

    Available versions for php:
        none (active)
        php70

sudo port select php php70

php -v

    PHP 7.0.3 (cli) (built: Feb  8 2016 04:09:22) ( NTS )
    Copyright (c) 1997-2016 The PHP Group
    Zend Engine v3.0.0, Copyright (c) 1998-2016 Zend Technologies
        with Zend OPcache v7.0.6-dev, Copyright (c) 1999-2016, by Zend Technologies

### 4.4. PHP 5.6

```sh
sudo port install php56 php56-apache2handler
sudo port install php56-cgi php56-gd php56-curl php56-iconv php56-gettext php56-mbstring php56-mcrypt php56-mysql php56-openssl php56-sockets php56-zip php56-opcache php56-sqlite
cd /opt/local/etc/php56
sudo cp php.ini-development php.ini
```

sudo vi /opt/local/apache2/conf/httpd.conf

    <IfModule mime_module>
    ...
    AddType application/x-httpd-php .php
    AddType application/x-httpd-php-source .phps
    </IfModule>

```sh
cd /opt/local/apache2/modules
sudo /opt/local/apache2/bin/apxs -a -e -n php5 mod_php56.so
```

sudo vi /opt/local/apache2/conf/httpd.conf

    DirectoryIndex index.php index.html
    
    # Include PHP configurations
    #Include conf/extra/mod_php70.conf
    Include conf/extra/mod_php56.conf

    $ sudo -i
    # cd /opt/local/etc/php56
    # cp php.ini php.ini.bak
    # defSock=$(/opt/local/bin/mysql_config --socket)
    # cat php.ini | sed \
      -e "s#pdo_mysql\.default_socket.*#pdo_mysql\.default_socket=${defSock}#" \
      -e "s#mysql\.default_socket.*#mysql\.default_socket=${defSock}#" \
      -e "s#mysqli\.default_socket.*#mysqli\.default_socket=${defSock}#" > tmp.ini
    # grep default_socket tmp.ini  # Check it!
    default_socket_timeout = 60
    pdo_mysql.default_socket=/opt/local/var/run/mysql56/mysqld.sock
    mysqli.default_socket=/opt/local/var/run/mysql56/mysqld.sock
    # mv tmp.ini php.ini
    # exit # OR rm php.ini.bak && exit

```sh
sudo port unload apache2
sudo port load apache2

export PATH=$PATH:/opt/local/bin
echo 'export PATH=$PATH:/opt/local/bin' ~/.profile
```

port select --list php

    Available versions for php:
        none (active)
        php56

sudo port select php php56

$ php -v

    PHP 5.6.20 (cli) (built: Apr  1 2016 21:57:30)
    Copyright (c) 1997-2016 The PHP Group
    Zend Engine v2.6.0, Copyright (c) 1998-2016 Zend Technologies
        with Zend OPcache v7.0.6-dev, Copyright (c) 1999-2016, by Zend Technologies



# How to Install Docker and Docker Compose on Ubuntu 20.04

在 Ubuntu 20.04 中安装 Docker 和 Docker Compose

## 安装 Docker

更新软件列表：

    $ sudo apt update

安装必备软件包：

    $ sudo apt install -y apt-transport-https ca-certificates curl software-properties-common

把 Docker 官方源 GPG key 添加到系统：

    $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

把 Docker 源添加到系统 apt 源中：

    $ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"

新添加的 apt 源，需要更新软件列表：

    $ sudo apt update

安装社区版 Docker `docker-ce`：

    $ sudo apt install docker-ce

检查 docker 服务是否已安装并正常运行：

    $ sudo systemctl status docker


## 安装 Docker Compose

先在[发布页面](https://github.com/docker/compose/releases)确认 Docker Compose 最新版本号，编写本 Howto 时版本为 `1.28.6`，下载对应平台的版本到本地：

    $ sudo curl -L "https://github.com/docker/compose/releases/download/1.28.6/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

更改文件权限，使其可执行：

    $ sudo chmod +x /usr/local/bin/docker-compose

确认是否安装成功：

    $ docker-compose --version


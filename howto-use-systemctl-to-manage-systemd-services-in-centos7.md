# How To Use Systemctl to Manage Systemd Services and Units

## 0. 简介

Systemd

## 1. 服务管理

### 1.1. 启动和停止服务

启动一个服务，很简单：

    sudo systemctl start _application_.service

`systemd` 会查找 `*.service` 文件中的管理命令，所以可以简写成如下的形式：

    sudo systemctl start _application_

停止一个服务：

    sudo systemctl stop _application_.service

### 1.2. 重启服务或重新加载配置

重启一个服务：

    sudo systemctl restart _application_.service

重新加载配置而不重启服务：

    sudo systemctl reload _application_.service

如果不确定该服务有没有 `reload` 功能用于重新加载配置，那么，`systemd` 为我们准备了一个 `reload-or-restart` 命令：

    sudo systemctl reload-or-restart _application_.service

### 1.3. 启用或禁用服务




### 1.4. 检查服务状态








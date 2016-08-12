# How To Install and Use Docker on CentOS 7

## 0. 简介

Docker


## 1. 系统需求

- 64-bit
- Kernel 3.10+

## 2. Docker 使用

### 2.1. 安装 Docker

安装 Docker：

```sh
sudo yum -y update
curl -fsSL https://get.docker.com/ | sh
```

启动 docker 服务：
```sh
sudo systemctl start docker
```

查看 docker 服务状态：
```sh
sudo systemctl status docker
```

将会看到类似如下输出：

    ● docker.service - Docker Application Container Engine
       Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; vendor preset: disabled)
       Active: active (running) since Wed 2016-08-10 12:08:33 CST; 21h ago
         Docs: https://docs.docker.com
     Main PID: 9139 (dockerd)
       Memory: 49.9M
       ...

启用 docker 服务，使其随机启动：
```sh
sudo systemctl enable docker
```

安装好 Docker 后，接下来就是使用 `docker` 命令去操作镜像和容器了，但 `docker` 命令默认是需要 root 权限的，也就是每次命令都要使用 `sudo` 前缀（如果你在 root 用户下去操作 docker 自然是没这个麻烦了）。我们也可以把用户添加到 docker 组里，docker 组是安装 docker 进自动创建的。

如果你不加 `sudo` 前缀执行 `docker` 命令的话，会看到如下错误提示：

    docker: Cannot connect to the Docker daemon. Is the docker daemon running on this host?.
    See 'docker run --help'.

要想摆脱 `sudo`，只需要把当前用户添加到 docker 组：

```sh
sudo usermod -aG docker $(whoami)
```

把其他用户添加到 docker 组：

```sh
sudo usermod -aG docker <username>
```

查看所有可用的子命令：

```sh
docker
```

    Usage: docker [OPTIONS] COMMAND [arg...]
           docker [ --help | -v | --version ]
    
    A self-sufficient runtime for containers.
    
    Options:
    
      --config=~/.docker              Location of client config files
      -D, --debug                     Enable debug mode
      -H, --host=[]                   Daemon socket(s) to connect to
      -h, --help                      Print usage
      -l, --log-level=info            Set the logging level
      --tls                           Use TLS; implied by --tlsverify
      --tlscacert=~/.docker/ca.pem    Trust certs signed only by this CA
      --tlscert=~/.docker/cert.pem    Path to TLS certificate file
      --tlskey=~/.docker/key.pem      Path to TLS key file
      --tlsverify                     Use TLS and verify the remote
      -v, --version                   Print version information and quit
    
    Commands:
        attach    Attach to a running container
        build     Build an image from a Dockerfile
        commit    Create a new image from a container's changes
        cp        Copy files/folders between a container and the local filesystem
        create    Create a new container
        diff      Inspect changes on a container's filesystem
        events    Get real time events from the server
        exec      Run a command in a running container
        export    Export a container's filesystem as a tar archive
        history   Show the history of an image
        images    List images
        import    Import the contents from a tarball to create a filesystem image
        info      Display system-wide information
        inspect   Return low-level information on a container, image or task
        kill      Kill one or more running container
        load      Load an image from a tar archive or STDIN
        login     Log in to a Docker registry.
        logout    Log out from a Docker registry.
        logs      Fetch the logs of a container
        network   Manage Docker networks
        node      Manage Docker Swarm nodes
        pause     Pause all processes within one or more containers
        port      List port mappings or a specific mapping for the container
        ps        List containers
        pull      Pull an image or a repository from a registry
        push      Push an image or a repository to a registry
        rename    Rename a container
        restart   Restart a container
        rm        Remove one or more containers
        rmi       Remove one or more images
        run       Run a command in a new container
        save      Save one or more images to a tar archive (streamed to STDOUT by default)
        search    Search the Docker Hub for images
        service   Manage Docker services
        start     Start one or more stopped containers
        stats     Display a live stream of container(s) resource usage statistics
        stop      Stop one or more running containers
        swarm     Manage Docker Swarm
        tag       Tag an image into a repository
        top       Display the running processes of a container
        unpause   Unpause all processes within one or more containers
        update    Update configuration of one or more containers
        version   Show the Docker version information
        volume    Manage Docker volumes
        wait      Block until a container stops, then print its exit code
    
    Run 'docker COMMAND --help' for more information on a command.

查看 docker 系统概况：

```sh
docker info
```

### 2.2. Docker 命令的使用

`docker` 命令格式如下：

    docker [option] [command] [arguments]



### 2.3. Docker 镜像

    docker run hello-world

    docker search centos

     STARS     OFFICIAL   AUTOMATED
     2526      [OK]       
     27                   [OK]
     13                   [OK]
     12                   [OK]
     11                   [OK]
     8                    [OK]
     4                    [OK]
     3                    [OK]
     3                    [OK]
     2                    [OK]
     1                    [OK]
     1                    [OK]
     1                    [OK]
     1                    [OK]
     0                    [OK]
     0                    [OK]
     0                    [OK]
     0                    [OK]
     0                    [OK]
     0                    [OK]
     0                    [OK]
     0                    [OK]
     0                    [OK]
     0                    [OK]
     0                    [OK]

    docker pull centos

    docker images




### 2.4. 运行 Docker 容器

    docker run -it centos /bin/bash

    docker commit -m "What did you do to the image" -a "Author Name" <name>/new_image_name

    docker images




### 2.5. 提交变更到 Docker 镜像

    exit




### 2.6. Docker 容器管理


    docker ps

    docker ps -a

    docker stop container-id


### 2.7. 推送 Docker 镜像到 Docker 仓库

    docker login -u docker-registry-username

    docker push docker-registry-username/docker-image-name

## 3. 搭建 Docker 私有仓库 Registry

### 3.1. 关于 Registry

官方的 Docker hub 是一个用于管理公共镜像的好地方，我们可以在上面找到我们想要的镜像，也可以把我们自己的镜像推送上去。但是，有时候，我们的使用场景需要我们拥有一个私有的镜像仓库用于管理我们自己的镜像。这个可以通过开源软件 Registry 来达成目的。

### 3.2. 搭建 Registry

Regitstry

## 4. 使用 Dockerfile 构建镜像


[我的 Dockefile](https://github.com/WisdomFusion/dockerfiles)

## 5. 总结




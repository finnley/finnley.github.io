+++
title = 'CentOS7安装Docker'
date = 2019-05-18T23:37:05+08:00
draft = true
categories = [ "Docker" ]
tags = [ "docker" ]
+++

[参考官方文档【Install Docker Engine on CentOS】](https://docs.docker.com/install/linux/docker-ce/centos/ "https://docs.docker.com/install/linux/docker-ce/centos/"). 

## Uninstall old versions

```
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

![](/images/docker/116.png)

<!-- more -->

## Install required packages

### SET UP THE REPOSITORY

```
$ sudo yum install -y yum-utils

$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

### INSTALL DOCKER ENGINE

```
$ sudo yum install docker-ce docker-ce-cli containerd.io
```

### Start Docker

```
$ sudo systemctl start docker
```

### Show version

```
$ sudo docker version
```

![](/images/docker/117.png)

### Verify that Docker Engine is installed correctly by running the hello-world image.

```
$ sudo docker run hello-world
```

![](/images/docker/118.png)

## Remove sudo

```
sudo docker image ls
```

每次使用docker命令的时候都要使用 `sudo` ,现在将 `sudo` 去掉，直接使用 `docker image ls` 指令,这其实就是让当前用户拥有执行docker指令的权限

1. 创建 docker 用户组，其实这个用户组在安装的时候就已经存在了

```
sudo groupadd docker
```

![](/images/docker/127.png)

2. 将当前用户添加到docker 的 group 中，vagrant 表示当前用户，docker 表示 docker的group

```
sudo gpasswd -a vagrant docker
```

![](/images/docker/128.png)

3. 这个时候去使用docker命令会发现还是会提示权限问题

```
docker image ls
```

![](/images/docker/129.png)

4. 重启 Docker 进程

```
sudo service docker restart
```

到这里再去执行docker命令还是会提示权限问题，我们需要退出，重新使用 `ssh` 连接

![](/images/docker/130.png)
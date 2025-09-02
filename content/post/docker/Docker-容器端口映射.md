+++
title = '容器端口映射'
date = 2020-04-29T22:19:17+08:00
draft = true
categories = [ "Docker" ]
tags = [ "docker" ]
+++


## 案例

比如创建一个网站，那么服务肯定要能够被别人访问到，如在本地创建了一个Nginx的服务

```
sudo docker run --name web -d nginx
```

<!-- more -->

![](https://images.notes.xuepincat.com/docker/port-mapping/1.png)

这样就创建了 Nginx 的 Container，现在通过 docker 创建了 nginx 的 Server

![](https://images.notes.xuepincat.com/docker/port-mapping/2.png)

但是现在 Nginx 的服务还访问不了，如果想访问只能通过下面的方式访问

```
sudo docker exec -it web /bin/bash
```

![](https://images.notes.xuepincat.com/docker/port-mapping/3.png)

如果需要访问，就需要将服务暴露给外面，其实NGINX的网络空间 netowork namespace 是独立的，它有自己的IP地址，NGINX 默认启用的端口是 80 端口，先看下 NGINX 的IP地址，默认是连接到 bridge 上面的

```
sudo docker network inspect bridge
```

![](https://images.notes.xuepincat.com/docker/port-mapping/4.png)

IP地址是 172.17.0.2，在主机上 PING下 下这个IP地址

```
ping 172.17.0.2
```

![](https://images.notes.xuepincat.com/docker/port-mapping/5.png)

发现是可以 PING 通的，因为主机上是有 docker0 这个 bridge 的,因为它是连到 docker0 这个 bridge 上面的

![](https://images.notes.xuepincat.com/docker/port-mapping/6.png)

## 访问 NGINX


访问NGINX 80 端口

### 方法一

```
telnet 172.17.0.2 80
```

![](https://images.notes.xuepincat.com/docker/port-mapping/7.png)

发现是可以访问的

### 方法二

```
curl http://172.17.0.2
```

此时会将 NGINX 的访问页面拉取下来

![](https://images.notes.xuepincat.com/docker/port-mapping/8.png)

在 docker host 里面是可以访问 NGINX 80 服务的，但是现在是要向外面提供服务的，也就是说现在不光要在 docker-node里面可以访问，也希望在外面也可以访问，这就要用到端口映射(port map)，现在NGINX的端口只绑定到了 172.17.0.2 这个IP的网络空间里面

现在需要将端口映射到本地 docker-node 这台机子上的80 端口上，就不需要访问 172.17.0.2 了，只需要 `curl 127.0.0.1` 就可以了

## 端口映射

现将NGINX关闭并删除

```
sudo docker stop web
```

```
sudo docker rm web
```

首先使用 `curl 127.0.0.1` 看下访问情况

![](https://images.notes.xuepincat.com/docker/port-mapping/11.png)

接着创建 NGINX 的 Container,指定参数 -p 表示端口映射，并将容器端口映射到本地端口

```
sudo docker run --name web -d -p 80:80 nginx
```

![](https://images.notes.xuepincat.com/docker/port-mapping/9.png)

```
sudo docker ps
```

![](https://images.notes.xuepincat.com/docker/port-mapping/10.png)

会看到 PORTS 这一列中看到 80 映射到 80/tcp,接着再去使用 `curl 127.0.0.1` 看下结果

![](https://images.notes.xuepincat.com/docker/port-mapping/12.png)

从图片中看到现在访问本地的80端口就已经可以访问到 Container 中的 NGINX 服务了，这也证明服务已经绑定到了本地 docker-node 的80端口上了

如果 docker-node 有一个自己的独立IP地址，比如在公网上面，docker-node有一个自己的公网地址，如果去访问docker-node的公网IP地址的时候是否就可以到容器中的NGINX了呢？

因为现在是通过 Vagrant 启动的docker-node,它的IP地址是在inett后面的，是个私有的IP地址

![](https://images.notes.xuepincat.com/docker/port-mapping/13.png)

现在在 Vagrantfile 中做一个端口映射（port map），将guest的80端口映射成opts里面的变量，比如docker-node1的80映射成8888，docker-node2映射成9999

![](https://images.notes.xuepincat.com/docker/port-mapping/14.png)

当前是在 node1 里面，将本地的80端口映射到了我笔记本的8888端口上,现在打来浏览器输入

```
127.0.0.1:8888
```

![](https://images.notes.xuepincat.com/docker/port-mapping/15.png)

现在就已经可以访问 NGINX 了，这个过程就是做了端口转发

## Container Port Map

![](https://images.notes.xuepincat.com/docker/port-mapping/16.png)

* 第一步

在Linux虚拟机里面通过 Docker 启动了一个 NGINX Container，NGINX的服务是绑定到了本地的Linux的network namespace 里面（172.17.0.2:80，这个IP：端口），Linux的虚拟机又是和docker0这个bridge连在一起的，通过 `-p` 实现了一个效果就是在本地的iptable里面添加了一个转换的规则，就是将访问本地（Linux虚拟机）80 端口的流量转发到NGINX Container的172.17.0.2:80端口上，这样当去访问linux虚拟机的本地80端口的时候就会将流量转发到NGINX里面，这样访问本地就可以访问NGINX的服务了

* 第二步

通过 Vagrantfile 做了一个端口的转发，因为在我们的电脑笔记本上无法直接访问192.168.205.10这个Linux虚拟机的地址的，我们可以做一个port map端口转发，当去访问我们笔记本127.0.0.1:8888的时候，就会将流量转发到vagrant创建的这台linux虚拟机的80端口上去，这样访问本地的8888端口就可以访问到container容器里面的NGINX了

如果不是通过vagrant创建的，而是通过阿里云，亚马逊云创建的一台云主机，就可以为云主机分配一个public 的IP地址，比如上面的192.168.205.10 就会对应变成一个公有的IP地址，任何人都可以访问的地址，通过上面的方式去云主机里面启动一个NGINX，然后将NGINX的80map到云主机的80，那么任何人都可以通过互联网访问NGINX的服务了，
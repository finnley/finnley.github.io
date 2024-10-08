+++
title = 'Docker bridge'
date = 2020-04-25T23:50:35+08:00
draft = true
categories = [ "Docker" ]
tags = [ "docker" ]
+++

在 docker 中两个container又是怎么连在一起的呢？另外在容器里面也是可以访问外部网络的比如访问百度，这些又是怎么实现的呢？

* bridge 

现在有两个运行中的容器，现将test2停掉并删除test2

```
sudo docker ps
sudo docker stop test2
sudo docker rm test2
```

<!-- more -->

![](https://images.notes.xuepincat.com/docker/bridge/1.png)

然后查看下test1的网络情况,下面命令会列举出这台机子上docker有哪些网络

```
sudo docker network ls
```

![](https://images.notes.xuepincat.com/docker/bridge/2.png)

图中看到有三个网络，它们有各自的Name和Driver

现在 test1 的容器是连接在bridge这个网络上的，可以通过下面命令查看网络详情

```
sudo docker network inspect 3c4fd8c0950c
```

![](https://images.notes.xuepincat.com/docker/bridge/3.png)

可以看到有一块 Containers ,可以看到Name为test1,还有MAC 地址和IP地址，这也说明test1的Container是连接到了 bridge 这个网络上面

在主机上使用 `ip a` 可以看到除了 loopback0, eth0, eth1 之外还有docker0,veth两个接口

![](https://images.notes.xuepincat.com/docker/bridge/4.png)

当输入下面命令时

```
sudo docker ps
```

![](https://images.notes.xuepincat.com/docker/bridge/5.png)

如果 Container 要连到docker0的bridge上面，docker0这个bridge它的network namespace 是本机，是linux的机器的network namespace,busybox又有自己的network namespace,这两个network namespace要连在一起，就需要一对veth,所以现在看到的第6个veth（红色框框）是负责连到docker0上面的

```
sudo docker exec test1 ip a
```

![](https://images.notes.xuepincat.com/docker/bridge/10.png)

本机的和容器里面的eth0其实是一对，所以Container test1 通过一对veth的pair连接到了主机的网络上面，具体是连接到了主机的docker0上面

如何验证veth是连接到了docker0上面呢？

首先安装,本机没有 `brctl` 这个命令，如果没有需要安装

```
sudo yum install bridge-utils
```

![](https://images.notes.xuepincat.com/docker/bridge/7.png)

```
brctl show
```

![](https://images.notes.xuepincat.com/docker/bridge/8.png)

可以看到当前linux机器上只有一个linux bridge 是 docker0,并且docker0有一个接口，是 vethd832a4c,上面图中本地的veth就是 vethd832a4c，这也就是说这个接口是连到的Linux Bridge 上的

现在再去创建一个 Container

```
sudo docker run -d --name test2 busybox /bin/sh -c "while true; do sleep 3600; done"
```

![](https://images.notes.xuepincat.com/docker/bridge/11.png)

执行下面命令

```
sudo docker network inspect bridge
```

![](https://images.notes.xuepincat.com/docker/bridge/12.png)

发现 Container 多了一个，除了一个 test1 之外，还有一个 test2 的容器连到了 bridge 上，它的地址是 172.17.0.3

接着在本机上运行 `ip a`

![](https://images.notes.xuepincat.com/docker/bridge/13.png)

发现又多了一个 veth,因为现在多了test2容器，又需要一对veth,类似一根线去连docker0

运行 `brctl show`

![](https://images.notes.xuepincat.com/docker/bridge/14.png)

会看到 docker0 连接了两个接口

![](https://images.notes.xuepincat.com/docker/bridge/15.png)

两边有两个容器，一个test1,一个test2 ,还有一个docker0的bridge,两个容器各通过一对 veth 分别连接到了 docker0 的 bridge 上面，所以说这两个 Container之间是可以通信的，这个也就有点像我们现实中的网络，比如家里的两台电脑同时连到一台路由器上，这两台机子是可以通信的，其实外面没有连 Internet, 它们之间也是一个局域网，之间也是可以通信的，图中原理也是和这个差不多的，两个容器只要通过 docker0 就可以相互通信了，两个容器分别是两个不同的 network namespace, 连接的docker0是在主机的一个默认的network namespace里面，通过两对线（两对接口）连接在一起

上面也就是两个容器之间是如何通信的

* 对于单个容器是如何访问 Internet?

![](https://images.notes.xuepincat.com/docker/bridge/16.png)

Linux主机是可以访问外网的，比如可以通过 eth0 这个接口出去访问 Internet,假如容器里面的数据包如果要访问外网，就要经过 docker0 这个bridge，如果能做一个NAT的话，网络地址转换，转换成 eth0 的地址，这时就会作为Linux主机的数据包发到外面去，自然而然就可以访问外网了，这里面就用了NAT网络地址转换的技术，这个技术又是通过 IP Tables实现的

上面就是容器间互访和容器如何进行外部通信
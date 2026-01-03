+++
title = '容器网络之host和none'
date = 2020-05-06T22:49:10+08:00
draft = true
categories = [ "Docker" ]
tags = [ "docker" ]
+++

## none network

通过 busybox 创建一个容器,并将容器连接到 none 类型的 network

```
sudo docker run -d --name test1 --network none busybox /bin/sh -c "while true; do sleep 3600; done"
```

<!-- more -->

![](https://images.notes.xuepincat.com/docker/network/1.png)

查看网络信息

```
sudo docker network inspect none
```

![](https://images.notes.xuepincat.com/docker/network/2.png)

会看到 none 连接了一个 Container 是test1,它的IP地址和Mac地址都没有

现在进到这个 test1的Container里面看下

```
sudo docker exec -it test1 /bin/sh
```

![](https://images.notes.xuepincat.com/docker/network/3.png)

进去之后运行 `ip a`

![](https://images.notes.xuepincat.com/docker/network/4.png)

图中可以看到除了有回环口外没有其他接口

没有其他接口就意味着test1所在的network namespace 是一个孤立的network namespace,也就是说除了通过 `exec` 的方式可以访问到这个容器以外就没有其他任何方式可以访问到这个容器了

既然不能访问它那还创建这种容器有什么作用呢？

有种可能是安全性比较高的容器，比如需要这个容器来存储密码之类的工具，不想让这个工具来让任何人访问，只需要在本地可以访问就行了，就可以通过这种方式比较安全一些

```
sudo docker stop test1
```

```
sudo docker rm test1
```

## host network

创建一个连接到 host network 的容器

```
sudo docker run -d --name test1 --network host busybox /bin/sh -c "while true; do sleep 3600; done"
```

```
sudo docker network inspect host
```

![](https://images.notes.xuepincat.com/docker/network/5.png)

图中可以看到 test1 是连接到host的network上面的，这个test1也是没有IP地址和Mac地址的

进入test1，运行 `ip a`

```
sudo docker exec -it test1 /bin/sh
```

![](https://images.notes.xuepincat.com/docker/network/6.png)

会发现和linux主机上是一样的


![](https://images.notes.xuepincat.com/docker/network/7.png)

也就是通过连接到 host 创建的容器没有自己独立的network namespace, 它是主机network namespace 共享一套，这也是在容器里面运行 `ip a` 和在外面运行是一样的

通过这种方式创建的容器有什么问题呢？

它既然是共享了主机的网络命名空间，就以为这端口可能会有冲突，比如想启动一个NGINX的Container，然后连接到 host 的network上，就只能启动一个，因为已经有一个绑定到本地的80端口了，就没法在启动第二个了


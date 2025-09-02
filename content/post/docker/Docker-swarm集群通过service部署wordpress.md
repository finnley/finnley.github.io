+++
title = 'Docker-集群通过service部署wordpress'
date = 2020-07-25T10:00:14+08:00
draft = true
categories = [ "Docker" ]
tags = [ "docker" ]
+++

在 swarm 里面创建 service 的时候，service 是创建在哪个节点上是不知道的，比如创建两个service,一个 wordpress 的 service,一个 mysql 的 service,这两个 servie 需要通信，这个两个容器有可能不在docker的节点上，那它们如何去通信呢？

可以通过 overlay 的方式让两个容器都连接到 overlay 的网络上，这样不管它们是否在同一台机器上，这两个容器都是可以相互通信的

1. 创建 overlay 的 network

在swarm-manager节点上创建overlay的网络driver,名字叫demo

```
docker network create -d overlay demo
```

```
docker network ls
```

![](https://images.notes.xuepincat.com/docker/swarm/24.png)

再到另外里那个 worker 节点行看下网络

```
docker network ls
```

![](https://images.notes.xuepincat.com/docker/swarm/25.png)

两个 worker 节点没有看到 demo 的网络

2. 创建 mysql service

```
docker service create --name mysql --env MYSQL_ROOT_PASSWORD=root --env MYSQL_DATABASE=wordpress --network demo --mount type=volume,source=mysql-data,destination=/var/lib/mysql mysql:5.7
```

`--name mysql`: 名字叫 mysql
`--env MYSQL_ROOT_PASSWORD=root`: 指定环境变量，数据库密码 
`--env MYSQL_DATABASE=wordpress`: 指定数据库
`--network demo`: 指定网络
`--mount type=volume,source=mysql-data,destination=/var/lib/mysql`: docker run 中是 `-v` 参数，service 中是 mount 参数，本地是 mysql-data,容器内部是 /var/lib/mysql
`mysql:5.7`: mysql image 名字

```
docker service ls
```

![](https://images.notes.xuepincat.com/docker/swarm/26.png)

查看下mysql信息

```
docker service ps mysql
```

![](https://images.notes.xuepincat.com/docker/swarm/27.png)

可以看到容器运行在了swarm-manager节点上，其实也有可能运行在其他节点行，比如 worker1 或者 worker2 节点行

也可以通过 `docker ps` 确认下

![](https://images.notes.xuepincat.com/docker/swarm/28.png)

3. 创建 wordpress service

```
docker service create --name wordpress -p 80:80 --env WORDPRESS_DB_PASSWORD=root --env WORDPRESS_DB_HOST=mysql --network demo wordpress
```

`-p 80:80`: 将 wordpress web 服务的80端口映射到外面的80

```
docker service pswordpress
```

![](https://images.notes.xuepincat.com/docker/swarm/29.png)

看到wordpress运行在了swarm-worker1的节点上

可以在swarm-worker1节点上输入 `docker ps` 确认下

![](http://images.notes.xuepincat.com/docker/swarm/30.png)

4. 访问 wordpress 服务

现在已经将wordpress的80端口暴露到外面的docker host的80端口了，但是现在wordpress运行在swarm-worker1的节点上

首选在 swarm-worker1 输入 `ip a` 查看下ip地址是 `192.168.205.11`,在浏览器中输入该IP地址

![](https://images.notes.xuepincat.com/docker/swarm/31.png)

![](https://images.notes.xuepincat.com/docker/swarm/32.png)

已经可以进行设置访问了，也就说明两个service是可以相互访问的

创建的 wordpress 分配运行在 swarm-worker1 的节点上，在浏览器中输入的地址也是 swarm-worker1 的地址，如果输入 swarm-manager (192.168.205.10) 的地址或者 swarm-worker2 (192.168.205.12) 的地址

![](https://images.notes.xuepincat.com/docker/swarm/33.png)

![](https://images.notes.xuepincat.com/docker/swarm/34.png)

会看到都是可以访问的

在创建 mysql 和 wordpress 之前先创建了名叫 demo 的 network,是 overlay 的 Driver 的 Network,但是在创建之后在 swarm-worker1和swarm-worker2的并没有看到有demo的网络产生，现在创建了wordpress，并且是在worker1上面，现在再去看下里面的网路

![](https://images.notes.xuepincat.com/docker/swarm/35.png)

会看到worker1上已经有了demo的网络了，这个过程就是swarm去完成的，当wordpress容器分配到swarm-worker1上之后，它要保持和manager进行通信，overlay的网路就需要同步，之前docker多机网络中overlay的network使用了第三方的外置的key-value存储DB `cted`,但在swarm模式下并不需要依赖key-value存储，它是由swarm自己去维护的，也就是swarm的机制它会去同步网络的创建，因为它要去实现多个机器之间的容器的通信肯定是要通过 overlay 的网络去实现
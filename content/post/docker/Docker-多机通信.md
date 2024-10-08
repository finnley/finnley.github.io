+++
title = '多机通信'
date = 2020-07-19T09:51:33+08:00
draft = true
categories = [ "Docker" ]
tags = [ "docker" ]
+++

![](https://images.notes.xuepincat.com/docker/network/8.png)

比如有两个容器分别在两个不容的机器上，两个机器本身是可以通信的，docker 除了有 bridge,none,host 之外还有另外一种网络 `Overlay`,可以创建一个叫 `Overlay` 的网络，来基于这个网络实现多机器通信

如果要实现多机器通信，还需要借助第三方分布式存储，比如创建了一个网络，图中左边部分，它的网段是 `172.17.0.0`, 掩码24位，如果在左边的机器创建一个容器，分配的地址是 `172.17.0.2`,然后要保证在右边机器上创建容器的时候 `172.17.0.2` 的地址不会被再用，比如会去分配一个 `172.17.0.3`,或者 `172.17.0.4` 之类的

那右边的机器是怎么知道会不会和之前的 `172.17.0.2` 的地址不一样的呢？

此时就需要一个分布式存储，通过这种存储就会知道 `172.17.0.2` 已经被用过了，`172.17.0.3` 没有被使用过就可以使用 `172.17.0.3`

这边选择的是一种开源的分布式存储 `etcd`

首先需要先搭建一个 `etcd` 的 cluster

## setup etcd cluster

#### docker-node1

* 下载

```
wget https://github.com/coreos/etcd/releases/download/v3.0.12/etcd-v3.0.12-linux-amd64.tar.gz
```

* 解压

```
tar zxvf etcd-v3.0.12-linux-amd64.tar.gz
```

* 进入解压目录

```
cd etcd-v3.0.12-linux-amd64/
```

* 启动一个进程

```
nohup ./etcd --name docker-node1 --initial-advertise-peer-urls http://192.168.205.10:2380 \
--listen-peer-urls http://192.168.205.10:2380 \
--listen-client-urls http://192.168.205.10:2379,http://127.0.0.1:2379 \
--advertise-client-urls http://192.168.205.10:2379 \
--initial-cluster-token etcd-cluster \
--initial-cluster docker-node1=http://192.168.205.10:2380,docker-node2=http://192.168.205.11:2380 \
--initial-cluster-state new&
```

* 查看

必须要在 etcd 的目录下执行

```
./etcdctl cluster-health
```

#### 在docker-node2 上

同样的在 docker-node2 上执行上面的一些步骤，注意启动进程的命令不一样

* 下载

```
wget https://github.com/coreos/etcd/releases/download/v3.0.12/etcd-v3.0.12-linux-amd64.tar.gz
```

* 解压

```
tar zxvf etcd-v3.0.12-linux-amd64.tar.gz
```

* 进入解压目录

```
cd etcd-v3.0.12-linux-amd64/
```

* 启动一个进程

```
nohup ./etcd --name docker-node2 --initial-advertise-peer-urls http://192.168.205.11:2380 \
--listen-peer-urls http://192.168.205.11:2380 \
--listen-client-urls http://192.168.205.11:2379,http://127.0.0.1:2379 \
--advertise-client-urls http://192.168.205.11:2379 \
--initial-cluster-token etcd-cluster \
--initial-cluster docker-node1=http://192.168.205.10:2380,docker-node2=http://192.168.205.11:2380 \
--initial-cluster-state new&
```

![](https://images.notes.xuepincat.com/docker/network/10.png)

#### 检查cluster状态

* docker-node1

```
./etcdctl cluster-health
```

* docker-node2

```
./etcdctl cluster-health
```

![](http://images.notes.xuepincat.com/docker/network/11.png)

#### 重启 docker 服务

目的是让docker知道要去使用 cluster_store

* docker-node1

```
sudo service docker stop
```

```
sudo /usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock --cluster-store=etcd://192.168.205.10:2379 --cluster-advertise=192.168.205.10:2375&
```

然后退出重新进入

```
exit
vagrant ssh docker-node1
```

此时 docker 服务会在后台运行

* docker-node2

```
sudo service docker stop
```

```
sudo /usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock --cluster-store=etcd://192.168.205.11:2379 --cluster-advertise=192.168.205.11:2375&
```

然后退出重新进入

```
exit
vagrant ssh docker-node2
```

#### 创建overlay network

首先看下 docker-node1 上 network

```
sudo docker network ls
```

再看下 docker-node2 上 netword

```
sudo docker network ls
```

![](https://images.notes.xuepincat.com/docker/network/13.png)

* 在 docker-node1 上创建一个demo的overlay network

```
sudo docker network create -d overlay demo
```

![](https://images.notes.xuepincat.com/docker/network/14.png)

上面的命令并没有在 docker-node2 中使用，此时再去 docker-node2 上查看下 network

```
sudo docker network ls
```

![](https://images.notes.xuepincat.com/docker/network/15.png)

会发现 docker-node2 上也有了个 demo,所以说两边是同步的，这个其实就是 etcd 做的

可以到 docker-node2 中 etcd 目录看下

```
./etcdctl ls /docker/nodes
```

![](https://images.notes.xuepincat.com/docker/network/16.png)

会看到存储了两个 node 的信息，etcd是key-value的database

再看下network

```
./etcdctl ls /docker/network/v1.0/network
```

![](https://images.notes.xuepincat.com/docker/network/17.png)

它有个id `0544fa9391a73d2e99db1d10964b1dfecc0a8d27022c23d100331fc42ad71c64`，在 docker-node1 上创建的 demo 的id是 `0544fa9391a7`

所以之前的 docker-node1 上创建的network会通过 etcd 的 docker-node2 发现并同时在node2上创建overlay的网络,这个demo的network是通过etcd从node1同步到node2的

* 在 overlay 网络之上创建 Container

在docker-node1上创建之前看下它的网络

```
sudo docker network inspect demo
```

![](https://images.notes.xuepincat.com/docker/network/18.png)

现在创建一个容器

```
sudo docker run -d --name test1 --net demo busybox /bin/sh -c "while true; do sleep 3600; done"
```

![](https://images.notes.xuepincat.com/docker/network/19.png)

此时如果也想在 docker-node2 上创建一个 test1 的容器

![](http://images.notes.xuepincat.com/docker/network/20.png)

会发现提示了这样一句话

`docker: Error response from daemon: endpoint with name test1 already exists in network demo.`

表示刚才在 docker-node1 上创建的 test1 的名字已经被使用了，因为这个名字已经存储到了 etcd 的 cluster 里面了，也就在第二个node2的节点上创建test1的时候回去 etcd 的key-value的store里面取查找，发现test1已经被使用了，所以就不能再使用了

可以使用 test2

```
sudo docker run -d --name test2 --net demo busybox /bin/sh -c "while true; do sleep 3600; done"
```

![](https://images.notes.xuepincat.com/docker/network/21.png)

接着看下test2的IP地址

```
sudo docker exec test2 ip a
```

![](https://images.notes.xuepincat.com/docker/network/22.png)

地址是 `10.0.0.3`

再去 node1 上看下test1的IP地址

```
sudo docker exec test1 ip a
```

![](https://images.notes.xuepincat.com/docker/network/23.png)

地址是 `10.0.0.2`

在看下网络,

```
sudo docker network inspect demo
```

![](https://images.notes.xuepincat.com/docker/network/24.png)

会看到 Container 有两个，一个地址是 test1 `10.0.0.2`,一个是 test2 `10.0.0.3`,这两个容器就是通过 docker 的 overlay 连接在了一起，并且它们是可以相互 ping 通的

在 test1 `10.0.0.2`上 ping `10.0.0.3`

```
sudo docker exec test1 ping 10.0.0.3
```

![](http://images.notes.xuepincat.com/docker/network/25.png)

也可以 ping 下 test2 的 hostname

```
sudo docker exec test1 ping test2
```

![](https://images.notes.xuepincat.com/docker/network/26.png)

之前创建了一个 demo,查看下网络

```
sudo docker network ls
```

发现多了一个 `docker_gwbridge`,

![](http://images.notes.xuepincat.com/docker/network/27.png)

查看下 test1 的 ip

![](http://images.notes.xuepincat.com/docker/network/28.png)

会看到有个IP `172.19.0.2`,这个其实是连接到 docker_gwbridge 上面的,这个网络主要是负责对外通信的

具体可以参考 `https://github.com/docker/labs/blob/master/networking/concepts/06-overlay-networks.md`

## 项目部署

* 在 node1 上部署 redis

* 在 node2 上运行 flask


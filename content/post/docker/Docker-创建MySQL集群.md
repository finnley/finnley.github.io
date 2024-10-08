+++
title = '创建MySQL集群'
date = 2018-09-23T16:51:47+08:00
draft = true
categories = [ "Docker" ]
tags = [ "docker" ]
+++

## PXC镜像安装

pxc集群比较特殊，只能安装在 `Linux` 系统之上，既可以在 `linux` 系统之上直接安装，也可以在 `docker` 上安装，这里选择的是后者，Docker的镜像仓库中包含了PXC数据库的镜像，下载即可。

[<span style="color:#FF1493;">https://hub.docker.com/r/percona/percona-xtradb-cluster/</span>](https://hub.docker.com/r/percona/percona-xtradb-cluster/)

![](/images/docker/34.png) 

在网页的右侧可以看到安装镜像的指令，这个指令执行之后，docker就会把镜像下载到docker虚拟机里面并安装

```
docker pull percona/percona-xtradb-cluster
```

或者

```
docker pull percona/percona-xtradb-cluster:5.7.21
```

另外一个方式就是本地安装，前提是本地存储了pxc镜像压缩文件，可以通过下面方式将镜像导入到docker虚拟机里面。其实安装的pxc镜像我们也可以通过保存到本地压缩文件，这样就可以拷贝到其他电脑上，如果想安装pxc镜像，直接就可以导入。

```
docker load < /home/soft/pxc.tar.gz
```

![](/images/docker/35.png)

执行 `docker images` 指令

![](/images/docker/36.png)

可能觉得名字很长，可以修改下名字

```
docker tag docker.io/percona/percona-xtradb-cluster pxc
```

或

```
docker tag percona/percona-xtradb-cluster:5.7.21 pxc
```

![](/images/docker/37.png)

再次执行下 `docker images` 指令,我们发现多了一个pxc的镜像

![](/images/docker/38.png)

现在我们可以把原先的镜像删除掉,然后再去查看一下镜像，发现没有了

```
docker rmi docker.io/percona/percona-xtradb-cluster 
```

或

```
docker rmi percona/percona-xtradb-cluster:5.7.21
```

![](/images/docker/39.png)

## 创建内部网络

出于安全考虑，需要给PXC集群实例创建Docker内部网络。上面安装的PXC镜像创建的容器就是PXC集群内部的数据库实例。

假如我们要搭建一个有5节点的集群，就要创建出5个PXC容器，创建出来的PXC容器不要直接对接Docker以外的网络，那样很不安全。出于安全考虑，我们需要先给PXC集群在docker虚拟机内部单独划分一个网段，这个网段外部是无法直接访问的，然后向外部开放什么端口，我们可以使用docker端口映射技术去实现。

### 指令

```
docker network create net1
```

创建名为 `net1` 的网段，这个网段的IP是什么样的呢？Docker虚拟机自带的网段是 `172.17.0.*`，这是它内置的网段，如果我们创建的网段是 `net1` ,那么我们的网址就是 `172.18.0.*`,如果再去创建第二个的网段，那么IP就是 `172.19.0.*`，如果我想创建的网段不是 `172.18.*.*`,我们也可以自己规定。

* 如果我创建的网段是 `net1` ,我想查看网段的相关信息，可以执行下面指令

```
docker network inspect net1
```

* 删除网段

```
docker network rm net1
```

### 实操

```
docker network create --subnet=172.18.0.0/24 net1
```

* 参数是 `--subnet=`，表示自己指定网段
* 自己指定的具体网段是 `172.18.0.0`
* 子网掩码是24位的
* 网段名字为net1

出现下面信息表示该网段已成功创建

![](/images/docker/40.png)

* 查看网段信息,其中的Subnet就是网段的具体IP

```
docker inspect net1
```

![](/images/docker/41.png)

* 如果将来要删除网段 

```
docker network rm net1
```

## 创建Docker卷

docker容器使用原则，一旦创建出docker容器，我们尽量不要在容器里面保存业务数据，要把业务数据保存到宿主机里面，使用的技术就是目录映射，把宿主机中的一个目录映射到容器内，运行容器的时候把业务数据保存到映射目录里面，也就是存储到了宿主机里面，这样如果出现什么故障的的话，宿主机有这个数据，我只需要把故障的容器删除掉，重新启动一个新的容器，然后把目录映射给新的容器，那么新的容器启动就自带了这些业务数据。

PXC运行在docker容器里面，它是无法直接使用映射的目录，也就是说我们如果采用映射目录的技术直接给PXC容器，PXC容器启动的时候会闪退，那我们就需要使用另外一种映射的技术，这个技术叫做 `Docker卷`。

### 实操

* 创建数据卷 v1

```
docker volume create --name v1
```

docker数据卷名字是通过 `--name` 参数来指定的，比如卷的名字叫 `v1` ,有时 `--name` 也可以不写，直接 `docker volume create v1`

我们创建的docker卷在宿主机上也是可以看得见的，它是在宿主机上的docker创建一个卷，这个卷在宿主机上是可以看到目录的，然后把这个卷映射给容器，这样我们PXC容器启动，就可以把数据映射到卷的目录里面，我们通过宿主机也可以看到映射的数据。

![](/images/docker/42.png)

上面是创建数据卷，提示数据卷 `v1` 已成功创建，下面是查看数据卷创建到什么位置

![](/images/docker/43.png)

从上面的信息可以看到数据卷在宿主机上真实的路径是 `/var/lib/docker/volumes/v1/_data` ，那么我们可以在将来创建PXC容器的时候将数据卷 `v1` 映射到容器的MySQL目录里面，如果我们想要删除这个数据卷也是可以的，使用下面指令

```
docker volume rm v1
```

## 创建PXC容器

### 指令

* 只需要向PXC镜像传入运行参数就能创建出PXC容器

在docker虚拟机上创建容器使用指令 `docker run` ,那么创建PXC容器也是 `docker run`,但是需要向指令传入一些特定的参数才能创建出我们想要的PXC容器


```
docker run -d -p 3306:3306
-v v1:/var/lib/mysql
-e MYSQL_ROOT_PASSWORD=abc123456
-e CLUSTER_NAME=PXC
-e XTRABACKUP_PASSWORD=abc123456
--privileged --name=node1 --net=net1 --ip 172.18.0.2
pxc
```

* -d: 创建出来的容器在后台去运行；
* -p: 端口的映射，前面的端口是宿主机的端口，后面的端口是容器的端口，就是把容器的3306端口映射到宿主机的3306端口上；
* -v: 路径的映射，之前我们创建了数据卷，v1就是我们刚才创建的数据卷，`v1:/var/lib/mysql` 表示v1数据卷映射MYSQL的数据目录，容器中的数据库目录是 `/var/lib/mysql` ；
* -e MYSQL_ROOT_PASSWORD=abc123456: 启动参数，表示创建出来的数据库实例对应的密码是什么，用户名就是root,我们改不了；
* -e CLUSTER_NAME=PXC: 表示创建出来的PXC集群的名字，名字可以自己去规定；
* -e XTRABACKUP_PASSWORD=abc123456: 表示数据库节点之间同步用到的密码；
* --privileged: 表示给到的最高权限；
* --name=node1: 给创建出来的容器起的名字，这里器的名字为node1；
* --net=net1: 表示使用的内部网段是net1；
* --ip 172.18.0.2: 容器分到的ip；
* pxc: 镜像的名字；

* 执行上面的指令，那么一个pxc的容器就成功创建出来了,这里我先不创建，我先创建数据卷 v2, v3, v4, v5

```
docker volume create v2
docker volume create v3
docker volume create v4
docker volume create v5
```

![](/images/docker/44.png)

* 创建pxc容器

```
docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=abc123456 -e CLUSTER_NAME=PXC -e XTRABACKUP_PASSWORD=abc123456 -v v1:/var/lib/mysql --privileged --name=node1 --net=net1 --ip 172.18.0.2 pxc
```

![](/images/docker/45.png)

容器的创建非常快，但是容器里面MySQL数据库的初始化并不是一瞬间就会完成的，MySQL数据库的初始化至少需要两分钟以上，数据库实例的初始化时比较耗时的，如果第一个PXC容器里的MySQL没有启动成功，我就创建了第二个PXC的容器，那么第二个PXC容器启动之后会闪退，因为第二容器的MySQL和第一个容器里面的MySQL做同步的时候，它会发现第一个容器里面的MySQL没有成功启动，那么它就会出现一些故障导致第二个PXC容器闪退，所以需要耐心等待第一个容器里MYSQL成功初始化，并且通过客户端能成功连接到MySQL实例了，再去创建第二个，第三个第四个第五个PXC实例。

我们打开Navicat客户端，创建一个MySQL连接，连接到第一个节点上名字为DB1,IP为宿主机的IP，端口是宿主机的端口，因为5个节点映射了5个端口是3306，用户名root

出现下面表示连接成功。

![](/images/docker/46.png)

上面操作成功后我们就可以创建第二个、第三个、第四个、第五个PXC节点的容器了

* 下面创建第二个PXC容器节点

```
docker run -d -p 3307:3306 -e MYSQL_ROOT_PASSWORD=abc123456 -e CLUSTER_NAME=PXC -e XTRABACKUP_PASSWORD=abc123456 -e CLUSTER_JOIN=node1 -v v2:/var/lib/mysql --privileged --name=node2 --net=net1 --ip 172.18.0.3 pxc
```

![](/images/docker/47.png)


注意只有第一个容器里的数据库实例初始化比较慢，其他的相对会比较快一点。

* 现在去创建第三个，第四个，第五个

```
docker run -d -p 3308:3306 -e MYSQL_ROOT_PASSWORD=abc123456 -e CLUSTER_NAME=PXC -e XTRABACKUP_PASSWORD=abc123456 -e CLUSTER_JOIN=node1 -v v3:/var/lib/mysql --privileged --name=node3 --net=net1 --ip 172.18.0.4 pxc
```

```
docker run -d -p 3309:3306 -e MYSQL_ROOT_PASSWORD=abc123456 -e CLUSTER_NAME=PXC -e XTRABACKUP_PASSWORD=abc123456 -e CLUSTER_JOIN=node1 -v v4:/var/lib/mysql --privileged --name=node4 --net=net1 --ip 172.18.0.5 pxc
```

```
docker run -d -p 3310:3306 -e MYSQL_ROOT_PASSWORD=abc123456 -e CLUSTER_NAME=PXC -e XTRABACKUP_PASSWORD=abc123456 -e CLUSTER_JOIN=node1 -v v5:/var/lib/mysql --privileged --name=node5 --net=net1 --ip 172.18.0.6 pxc
```

![](/images/docker/48.png)

* 这样5个PXC容器就都启动成功了，可以通过客户端连接一下容器里面的MYSQL

![](/images/docker/49.png)

* 我们在 `DB1` 中创建一个 `test` 的数据库

![](/images/docker/50.png)

* 我们看一下其他节点

![](/images/docker/51.png)

* 我们在 `DB1` 节点的 `test` 库中创建一个数据表，表名为 `student` 学生表，并往表中插入一条数据

![](/images/docker/52.png)

* 我们提交之后看一下其他的节点

节点2:

![](/images/docker/53.png)

节点3:

![](/images/docker/54.png)

节点4:

![](/images/docker/55.png)

节点5:

![](/images/docker/56.png)

* 现在我在节点5中再插入一条数据，看一下是否会同步到其他节点

![](/images/docker/57.png)

节点1:

![](/images/docker/58.png)

节点2:

![](/images/docker/59.png)

节点3:

![](/images/docker/60.png)

节点4:

![](/images/docker/61.png)




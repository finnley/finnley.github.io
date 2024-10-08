+++
title = '搭建Redis集群'
date = 2018-11-18T19:04:13+08:00
draft = true
categories = [ "Docker" ]
tags = [ "docker" ]
+++

## 高速缓存介绍

- 高速缓存利用内存保存数据，读写速度远超硬盘
- 高速缓存可以减少I/O操作，降低I/O压力

比如在微信里发红包，一群人抢红包，发红包抢红包就用到了告诉缓存，如果不使用告诉缓存，发红包这条记录就要写入到数据库中，其他人去抢红包的时候发起抢红包的请求，服务器就要去读取数据库的这条红包记录，然后大家一起瓜分红包的金额，试想象下，春节期间几亿人发红包几亿人抢红包，就会对数据库压力非常大，如果真把发红包抢红包的记录写到数据库中，给用户的体验就不太好。如果使用高速缓存，发的红包保存到高速缓存中，其他人在抢红包的时候发起请求，服务器只需要到高速缓存里读写数据就可以了，不需要走数据库，减少了很多的I/O操作。

再比如打开手机淘宝APP,有些数据就是从缓存里面加载的，有些数据是从数据库中加载的，比如有些促销秒杀的热门商品，就可以将它们保存到高速缓存中，在加载的时候速度还是比较快的，另外淘宝还有基于大数据的内容推荐商品，这些商品并不是共性的商品，只是我们个人喜欢的商品，这些商品就可以从数据库中读取，所以淘宝首页的展示信息一部分是从告诉缓存中读取的，有一部分是从数据库中读取的。当然高速缓存不一定是 Redis,也可以是其他的产品。

## Redis介绍

- Redis是Vmware开发的开源免费KV型NoSQL缓存产品
- Redis具有很好的性能，最多可以提供10万次/秒的读写
- 目前新浪微博团队组建了世界上最大规模的Redis集群

## Redis集群介绍

Redis目前的集群方案分为以下几种：

1. RedisCluster: 官方推荐，没有中心节点

> 什么叫没有中心节点？

之前使用的pxc集群，pxc集群就没有中心节点，比如数据库集群里面有一部分节点挂掉了，那么剩余的节点如果超过半数，就要选出一个主节点，其他节点和这个主节点进行同步数据，注意这个主节点不是中心节点，这个主节点只是说保存的数据是最新的并且保存的数据最多，集群中剩余的节点就跟主节点同步数据，同步数据之后主节点就自动消失了，集群中就没有主节点了，所以主节点不是中心节点，这个RedisCluster类似。

2. Codis: 中间件产品，存在中心节点

Codis是360公司推出的Redis集群方案，它是存在中心节点的，中心节点一旦挂掉之后，整个Redis集群就不能用了

3. Twemproxy: 中间件产品，存在中心节点

也是中心节点一旦挂掉之后，整个Redis集群就不能用了

## RedisCluster

- 无中心节点，客户端与Redis集群中任何节点直连，不需要中间代理层， Redis集群中每个节点都是可读可写的

- 数据可以被分片存储

RedisCluster存储数据是分片存储的，就是每一个节点存储的数据都是不一样的，假设现在Redis集群有三个节点A，B，C，现在我存储很多数据到集群中，A，B，C三个节点存储的数据都是不一样的，也就是要存储的数据要被切分存储到不同的Redis节点上面，如果这种情况下ABC任何一个节点挂掉，是不是就损失了一部分数据呢，对，确实是这样子的。

为了避免这种情况，就需要为ABC三个节点分别设置冗余节点，就是备份，A节点挂掉之后有冗余节点可以使用，所以不会出现A节点挂掉之后数据就没有了，冗余节点可以继续的提供A节点的服务，此时数据还在。

- Redis集群管理方便，后续可自行增加或摘除已存在节点

### RedisCluster 示意图

![](/images/docker/112.png)

上图中使用了三节点Redis构成了一个Redis集群，但是为了避免某一个节点出现故障数据丢失，又为三个节点配置了冗余节点，所以算下来整个RedisCluster用到了6个节点的Redis,我们正常执行业务操作的Redis节点叫做Master节点，也叫做主节点，与Master节点进行数据同步的节点叫做Slave节点，也叫做从节点。

## 主从同步

* Redis集群中的数据库复制是通过主从同步来实现的

* 主节点（Master）把数据分发给从节点（Slave）

![](/images/docker/227.png)

Redis 集群数据采用的是切分存储，也就是集群中每一个Redis节点存储的数据都不相同，一旦Redis节点挂掉，那么它存储的数据也就丢失了，为了避免这种情况就需要引入冗余节点，Master和Slave节点之间的数据是实时同步的。当Master节点挂掉之后，可以使用Slave节点继续开展业务，这就是主从同步的好处。

## Redis集群高可用

* Redis集群中应该包含奇数个Master，至少应该有3个Master

因为Redis集群和PXC集群都有选举机制，当集群中的大部分节点挂掉了，也就是说超过一半节点挂掉，剩余的节点是无法进行选举然后组成新的集群的。如果采用两个Master节点去构建一个集群，如果一个Master挂掉，那么剩余的节点数量没有出超过一半，那么Redis集群就不可用了，如果三个节点的Master，挂掉一个还剩两个，还可以进行选举，还可以组成一个新的集群。

* Redis集群中每个Master都应该有Slave

如果客户端连接到第一个Master,并向第一个Master写入数据，数据应该切分到哪个Master节点上，如果就是存储到第一个Master节点上就进行保存，保存后就会到对应的Slave节点。如果经过计算发现数据不应该存储到当前的Master(第一个)，那么这个Master就会把数据发送到对应的Master存储，然后经过复制对应的Slave也会同步数据，所以在Redis集群里面通过客户端连接任意任何Master节点都可以读写数据，不管数据应该保存到这个Master上还是其他的Master上，正常向这个Master写入数据就可以了。

## 配置RedisCluster集群

* 安装Redis镜像

```
docker pull yyyyttttwwww/redis
docker tag docker.io/yyyyttttwwww/redis redis
```

* 创建net2网络

```
docker network create --subnet=172.19.0.0/16 net2
```

* 创建Redis容器

```
docker run -it -d --name r1 -p 5001:6379 --net=net2 --ip 172.19.0.2 redis bash
```

这里并不是容器启动了，redis就启动，还要进入容器进行设置才能启动redis

* 进入容器

```
docker exec -it r1 bash
```

* 配置Redis节点

进入容器后需要修改Redis配置文件，修改Redis配置文件后Redis才可以启动，因为Redis默认是关闭了RedisCluster集群功能，然后启动redis才能加入到集群里

配置文件所在路径 `/usr/redis/redis.conf`

```
daemonize yes                   #以后台进程运行
cluster-enabled yes             #开启集群
cluster-config-file nodes.conf  #集群配置文件,一旦搭建起RedisCluster集群，集群里的一些节点信息保存在什么文件里面，文件名随便取，这里取的nodes.conf
cluster-node-timeout 15000      #超时时间 集群内部节点和节点连接超时时间
appendonly yes                  #开启AOF模式 Redis开启日志功能 一旦宕机可以通过AOF日志恢复Redis数据
```

* 打开配置文件

```
vi /usr/redis/redis.conf
```

* 找到上面的配置项并修改保存退出

* 启动Redis,首先进入Redis所在目录

```
cd /usr/redis/src
./redis-server ../redis.conf
```

到这里第一个redis节点就已经启动成功

## 创建r2节点

```
docker run -it -d --name r2 -p 5002:6379 --net=net2 --ip 172.19.0.3 redis bash
#进入r2节点
docker exec -it r2 bash
#配置
vi /usr/redis/redis.conf
cd /usr/redis/src
./redis-server ../redis.conf
```

## 创建r3节点

```
docker run -it -d --name r3 -p 5003:6379 --net=net2 --ip 172.19.0.4 redis bash
#进入r3节点
docker exec -it r3 bash
#配置
vi /usr/redis/redis.conf
cd /usr/redis/src
./redis-server ../redis.conf
```

## 创建r4节点

```
docker run -it -d --name r4 -p 5004:6379 --net=net2 --ip 172.19.0.5 redis bash
#进入r4节点
docker exec -it r4 bash
#配置
vi /usr/redis/redis.conf
cd /usr/redis/src
./redis-server ../redis.conf
```

## 创建r5节点

```
docker run -it -d --name r5 -p 5005:6379 --net=net2 --ip 172.19.0.6 redis bash
#进入r5节点
docker exec -it r5 bash
#配置
vi /usr/redis/redis.conf
cd /usr/redis/src
./redis-server ../redis.conf
```

## 创建r6节点

```
docker run -it -d --name r6 -p 5006:6379 --net=net2 --ip 172.19.0.7 redis bash
#进入r6节点
docker exec -it r6 bash
#配置
vi /usr/redis/redis.conf
cd /usr/redis/src
./redis-server ../redis.conf
```

## 安装redis-trib.rb

```
cp /usr/redis/src/redis-trib.rb /usr/redis/cluster/
cd /usr/redis/cluster
apt-get install ruby
apt-get install rubygems
gem install redis
```

## 创建Redis集群

```
docker exec -it r1 bash
cd /usr/redis
mkdir cluster && cd cluster
cp src/redis-trib.rb ./cluster/
cd cluster
./redis-trib.rb create --replicas 1 172.19.0.2:6379 172.19.0.3:6379 172.19.0.4:6379 172.19.0.5:6379 172.19.0.6:6379 172.19.0.7:6379
```

--replicas 1参数表示为每个主节点创建一个从节点

![](/images/docker/228.png)

到这里redis集群已经成功创建出来了，下面通过客户端去连接集群，有6个redis节点，它们都含有redis客户端，所以可以任意访问一个redis节点，下面访问的r1节点

## 连接集群

* 进入r1容器

```
docker exec -it r1 bash
```

* 连接

```
/usr/redis/src/redis-cli -c
```

-c 表示连接redis-cluster

* 测试

```
set a 10
get a
```

![](/images/docker/229.png)

经过redis计算，这个数据存入到了 `172.19.0.4` 这个节点上

这个redis集群是由6个节点组成，3个Master，3个Slave，经过计算这个 `172.19.0.4` 应该是第三个 Master,因为当前的节点是 `172.19.0.2`

* 挂掉 `172.19.0.4` Master节点，由Slave节点接替Master

```
docker pause r3
```

* 退出刚才的客户端，因为连接的r3的节点，所以需要先退出，然后再通过客户端连接redis集群

```
exit
/usr/redis/src/redis-cli -c
```

![](/images/docker/230.png)

会看到这个节点 `172.19.0.4` 已经挂掉了，但是没有关系，这个节点之前是Master,现在还有 Slave 节点可以使用，这个 slave 节点是 `172.19.0.7`，之前是 Slave，现在已经升级成 Master 节点了

* 读取数据测试

```
get a
```

![](/images/docker/231.png)

成功读取。

* 将 r3 节点恢复

```
docker unpause r3
```

重新退出然后连接redis，执行 `cluster nodes`

![](/images/docker/232.png)

会看到 `172.19.0.4` 这个节点被降级到 salve 了
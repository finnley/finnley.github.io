+++
title = '构建Redis镜像'
date = 2021-05-04T15:38:18+08:00
draft = true
categories = [ "Docker" ]
tags = [ "docker" ]
+++

## redis.conf

到这个网站下下载对应版本的 redis.conf 文件

https://raw.githubusercontent.com/antirez/redis/6.2/redis.conf

## 修改配置文件

```
daemonize yes                   #以后台进程运行
cluster-enabled yes             #开启集群
cluster-config-file nodes.conf  #集群配置文件,一旦搭建起RedisCluster集群，集群里的一些节点信息保存在什么文件里面，文件名随便取，这里取的nodes.conf
cluster-node-timeout 15000      #超时时间 集群内部节点和节点连接超时时间
appendonly yes                  #开启AOF模式 Redis开启日志功能 一旦宕机可以通过AOF日志恢复Redis数据
```

## Dockerfile

* 构建目录

```
mkdir redis
wget https://download.redis.io/releases/redis-6.2.3.tar.gz?undefined&undefined
vim Dockerfile
```

* 编写 Dockfile

```
FROM ubuntu

ADD redis-6.2.3.tar.gz /
RUN mv /redis-6.2.3 /usr/redis
ADD redis.conf /usr/redis

RUN apt update -y && apt install -y gcc make

WORKDIR /usr/redis
RUN make

RUN apt remove --purge -y gcc make

VOLUME ["/var/log/redis/", "/data"]

EXPOSE 6379
```

* 构建镜像

```
docker build -t finnley/redis .
```

## 创建容器

```
docker run -it -d --name r111 -p 5008:6379 --net=net2 --ip 172.19.0.8 finnley/redis bash
```

* 进入容器启动 redis 

```
docker exec -it r111 bash
cd /usr/redis/src
./redis-server ../redis.conf
```

* 同理，分别创建另外5个容器，并启动

## 创建集群

进入其中一个容器创建集群

```
redis-cli --cluster create 172.19.0.8:6379 172.19.0.9:6379 172.19.0.10:6379 172.19.0.11:6379 172.19.0.12:6379 172.19.0.13:6379 --cluster-replicas 1
```

![](/images/docker/233.png)

后面的测试步骤和之前一样。
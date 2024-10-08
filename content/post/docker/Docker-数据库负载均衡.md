+++
title = '数据库负载均衡'
date = 2018-09-24T00:18:30+08:00
draft = true
categories = [ "Docker" ]
tags = [ "docker" ]
+++

## 数据库负载均衡

在PXC集群中，任何一个数据节点都是可以读写的，一旦PXC集群上线之后，我们不能把所有的数据库请求全部发送给一个数据库节点，PXC集群里那么多数据节点，都应该去参加数据请求的处理，如果要实现数据请求均匀的发送给每一个数据节点，这个技术就叫做数据库的负载均衡。

## 为什么要负载均衡（必要性）

* 虽然搭建了集群，但是不使用数据库负载均衡，单节点处理所有请求，负载高，性能差

![](/images/docker/62.png)

如果不使用负载均衡，应用程序会把所有的请求都发送给一个PXC节点，这个节点负载特别高，所以很容易出现崩溃，而集群里其他的节点又特别空闲，这种情况就特别不好，所以在结构上要做一些调整。

* 使用Haproxy做负载均衡，请求被均匀分发给每个节点，单节点负载低，性能好

![](/images/docker/63.png)

经过调整之后，引入了新的技术，这个技术叫 `Haproxy`,它是一个老牌的中间件产品，'Haproxy' 不是数据库，它只是一个转发器而已。

应用程序发送请求，把这个请求发送给Haproxy，由Haproxy把这些请求转发给具体数据库节点上，这样就可以把大量的请求均匀地发送给每一个PXC节点，这样就不会让某一个PXC处理大量的请求，而是我们集群中的每一个节点去共同承担共同处理数据的请求，这样每一个节点的负载都比较低，这样的架构才是比较合理的架构。

## 下载镜像

* Docker仓库中保存了 `Haproxy` 的镜像，只需要下载即可

```
docker pull haproxy
```

![](/images/docker/64.png)

* 我们查看一下镜像 `docker images`

```
docker images
```

![](/images/docker/65.png)

## 创建Haproxy配置文件

安装了 `Haproxy` 镜像先不要着急去创建容器，`Haproxy` 里面是不包含配置文件的,需要在宿主机上创建一个 `haproxy.cfg` 的配置文件，存放目录可以自己去指定，然后需要使用目录映射技术把 `soft` 目录映射到 `haproxy` 容器里面，那么在`haproxy` 容器就可以知道这个配置文件，然后启动 `Haproxy` 的服务。

```
touch /home/soft/haproxy/haproxy.cfg
```

配置文件强请参考 [<span style="color:#FF1493;">https://zhangge.net/5125.html</span>](https://zhangge.net/5125.html) 

我这里提供的配置文件内容：

```
global
    #工作目录
    chroot /usr/local/etc/haproxy
    #日志文件，使用rsyslog服务中local5日志设备（/var/log/local5），等级info
    log 127.0.0.1 local5 info
    #守护进程运行
    daemon

defaults
    log	global
    mode	http
    #日志格式
    option	httplog
    #日志中不记录负载均衡的心跳检测记录
    option	dontlognull
    #连接超时（毫秒）
    timeout connect 5000
    #客户端超时（毫秒）
    timeout client  50000
    #服务器超时（毫秒）
    timeout server  50000

#监控界面	
listen  admin_stats
    #监控界面的访问的IP和端口
    bind  0.0.0.0:8888
    #访问协议
    mode        http
    #URI相对地址
    stats uri   /dbs
    #统计报告格式
    stats realm     Global\ statistics
    #登陆帐户信息
    stats auth  admin:abc123456

#数据库负载均衡
listen  proxy-mysql
    #访问的IP和端口
    bind  0.0.0.0:3306  
    #网络协议
    mode  tcp
    #负载均衡算法（轮询算法）
    #轮询算法：roundrobin
    #权重算法：static-rr
    #最少连接算法：leastconn
    #请求源IP算法：source 
    balance  roundrobin
    #日志格式
    option  tcplog
    #在MySQL中创建一个没有权限的haproxy名字的账户，密码为空。Haproxy使用这个账户对MySQL数据库心跳检测
    option  mysql-check user haproxy
    server  MySQL_1 172.18.0.2:3306 check weight 1 maxconn 2000  
    server  MySQL_2 172.18.0.3:3306 check weight 1 maxconn 2000  
    server  MySQL_3 172.18.0.4:3306 check weight 1 maxconn 2000 
    server  MySQL_4 172.18.0.5:3306 check weight 1 maxconn 2000
    server  MySQL_5 172.18.0.6:3306 check weight 1 maxconn 2000
    #使用keepalive检测死链
    option  tcpka  
```

## 创建Haproxy容器

### 指令

```
docker run -it -d
-p 4001:8888 -p 4002:3306 
-v /home/soft/haproxy:/user/local/etc/haproxy
--name haproxy --privileged --net=net1
haproxy
```

* 端口 3306：haproxy对外提供的数据库负载均衡的服务公开的端口就是3306，我需要将端口映射到宿主机，宿主机上的3306端口已经被占用了，所以这里选择4002端口；
* 端口 8888：haproxy提供了一个后台监控的画面，这个监控的画面在配置文件里面定义的就是8888，然后8888映射到宿主机上的端口这里选择的是4001端口，所以将来通过4001宿主机端口就可以进入到haproxy提供的监控画面上，就可以看到数据库负载均衡的状态；
* -v /home/soft/haproxy:/user/local/etc/haproxy：目录的映射，把宿主机 `/home/soft/haproxy` 映射到容器 `/user/local/etc/haproxy` 目录中，将来在宿主机上的目录中放置配置文件，容器中的目录中也就可以看到这个配置文件了；
* --name haproxy：给容器起的名字；
* --privileged：权限；
* --net=net1：网段；
* haproxy：镜像的名字；

上面指令执行完后就代表容器成功启动了，进入容器中执行下面指令

```
haproxy -f /usr/local/etc/haproxy/haproxy.cfg
```

然后进入监控画面就可以看到数据库运行状态了.

* 将haproxy配置文件上传到服务器

![](/images/docker/66.png)

* 创建haproxy容器

```
docker run -it -d -p 4001:8888 -p 4002:3306 -v /home/soft/haproxy:/usr/local/etc/haproxy --name h1 --privileged --net=net1 --ip 172.18.0.7 haproxy
```

![](/images/docker/67.png)

创建的容器在后台运行，我必须进入后台运行的容器，执行命令，把haproxy中间件启动起来，然后才能实现数据库的负载均衡

* 下面指令表示进入后台运行的容器

```
docker exec -it h1 bash
```

* -it:交互方式，有界面
* bash: 运行指令为bash

![](/images/docker/68.png)

* 启动haproxy指令

```
haproxy -f /usr/local/etc/haproxy/haproxy.cfg
```

* -f:加载配置文件
* /usr/local/etc/haproxy/haproxy.cfg: 配置文件位置

在DB数据库中创建一个haproxy的账号密码为空，随便是DB1,还是其他的DB几。因为是haproxy中间件，要用这个账号登录数据库然后发送消息检测，`%` 表示任何 `ip` 都可以以这个账号登录数据库,该语句我是在DB1数据库中执行的

```
CREATE USER 'haproxy'@'%' IDENTIFIED BY '';
```

![](/images/docker/69.png)

打开浏览器，地址IP是宿主机的IP，端口是 `4001`，相对路径是 `dbs`,回车之后需要数据账号密码，账号密码配置文件里面都有配置，这里账号是 `admin`,密码是 `abc123456`

```
http://IP:4001/dbs
```

登录用户名和密码在配置文件中有写

![](/images/docker/71.png)

下图可以看到一共是5个数据库节点，分别是MYSQL_1,MYSQL_2,MYSQL_3,MYSQL_4,MYSQL_5,现在5个节点都是畅通运行的，待会我会把某一个节点挂掉，会发现背景色就不是绿色的了

* 停止 `node1` 节点

```
docker stop node1
```

![](/images/docker/72.png)

切换到后台管理画面，会发现第一个背景色变成了其他颜色，表示这个节点已经出现了故障，其他的四个节点还可以正常运行，正常向haproxy发送数据库的请求并不会受到影响，因为haproxy会把请求分发给正常运行的MySQL节点

![](/images/docker/73.png)

* 在MySQL客户端上连接haproxy，密码是 `abc123456`

![](/images/docker/74.png)

其实Haproxy并不存储任何数据，我把请求发送给Haproxy，它只是把请求转发给真实的数据库实例

* 验证，我向H1节点的数据库student表中插入C的数据，然后保存提交

![](/images/docker/75.png)

提交的数据要么被轮询发送给DB2,要么分发给DB3，要么分发给DB4，要么分发给DB5，这次发给你，下次就会发给我，假如发给DB4了，因为PXC有同步机制，它会同步到其他的数据库节点

* 我打开DB2

![](/images/docker/76.png)

* 再打开DB5

![](/images/docker/77.png)

DB1就不要看了，因为已经挂掉了。

所以我们正常去操作haproxy，向它传递增删该查的语句，它都会均匀的分发给真实的MySQL实例，所以它起到一个负载均衡的作用，不会让你的数据库请求全部集中到一个数据库节点上去执行，它把你的请求均匀的分散出去，那么每一个数据库实例它得到的请求数量就小很多了。

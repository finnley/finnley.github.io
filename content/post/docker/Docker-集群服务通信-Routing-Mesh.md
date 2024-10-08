+++
title = '集群服务通信-Routing Mesh'
date = 2020-07-25T12:41:57+08:00
draft = true
categories = [ "Docker" ]
tags = [ "docker" ]
+++

有两个service,一个 mysql,一个wordpress,通过service的创建，将它们部署到了swarm cluster里面，这两个service位于不同的节点

这个两个service之间是可以相互通信的，并且它们之间是可以通过service name进行通信，这里就涉及到DNS服务发现的概念

![](https://images.notes.xuepincat.com/docker/swarm/37.png)

通过 docker compose 部署 application的时候，application里面的service如果连接到了一个网络上，它们之间是可以通过 service name qu 相互访问的，这个底层是通过DNS服务实现的

swarm 有内置一个DNS服务发现功能，在创建一个service的时候，并且是连接在overlay的网路上面，就会为连接在这个overlay网络上面的所有service增加一条DNS的记录，然后通过这条DNS记录就能知道IP地址，然后就可以访问服务了

实际上DNS的name就是service的name,但是DNS记录里面的IP并不是实际上service所在的容器IP，而是有个虚拟的IP（VIP），比如service有很多横向扩展scale,对于service，因为有很多扩展，不定在哪台机器上有哪个进程宕调，就可能会在某台机器上重新启动一个，这个时候的IP地址就变了，或者在scale的时候有很多service,就会对应很多的IP地址，如果在DNS里面通过每个service的容器的地址去标识是很不稳定的，因为IP地址是在变化的，针对这种情况就可以通过虚拟VIP来解决这种问题，会为service分配虚拟IP，这个IP是不会变的，一旦service创建之后IP就不会变了，但是这个IP背后所指的实际的IP，也就是service会在某个机器上创建容器，这个容器的具体IP地址，这个是通过IOES实现的

1. 创建一个名叫 demo 的 overlay network

```
docker network create -d overlay demo
```

2. swarm-manager 节点上创建service

```
docker service create --name whoami -p 8000:8000 --network demo -d jwilder/whoami
```

`jwilder/whoami`: docker image,提供web服务，访问8000端口会返回容器host name

```
docker service ls
```

```
docker service ps whoami
```

![](https://images.notes.xuepincat.com/docker/swarm/38.png)

可以看到service运行在 swarm-worker2 的机器上

可以到 swarm-worker2 上使用 `docker ps` 查看下

![](https://images.notes.xuepincat.com/docker/swarm/39.png)

监听了本地的8000端口，使用 `curl 127.0.0.1:8000` 看下结果

![](https://images.notes.xuepincat.com/docker/swarm/40.png)

会看到返回了 service 的host name,到这里第一个service 就创建好了

3. 再在 swarm-manager 节点上创建一个名为client的service连接到demo网络上

```
docker service create --name client -d --network demo busybox sh -c "while true;do sleep 3600;done"
```

```
docker service ls
```

```
docker service ps client
```

![](https://images.notes.xuepincat.com/docker/swarm/41.png)

可以看到 client 的service位于swarm-worker1的节点上

进入 swarm-worker1 的节点通过 `docker ps` 查看下

![](https://images.notes.xuepincat.com/docker/swarm/42.png)

4. 进入service创建的容器里面

```
docker exec -it 86c402920432
```

![](https://images.notes.xuepincat.com/docker/swarm/43.png)

5. 在client的service中ping下whoami的service name

```
ping whoami
```

![](https://images.notes.xuepincat.com/docker/swarm/44.png)

发现是可以ping通的，但是实际上ping的却是 `10.0.1.120` 这个IP,当前节点是 swarm-worker1的节点上，而 whoami 的节点是在 swarm-worker2的节点上，也就是说可以通过 service name 访问到 whoami 的service, `10.0.1.120` 这个IP地址并不是swarm-worker2上运行的whoami的service地址

6. 在swarm-manager节点上通过 scale 将whoami的service横向扩展成2个

```
docker service scale whoami=2
```

![](https://images.notes.xuepincat.com/docker/swarm/45.png)

会看到有一个service部署到了swarm-manager上

进入到 swarm-worker2 节点上通过 `docker ps` 查看下

![](http://images.notes.xuepincat.com/docker/swarm/46.png)

7. 再到swarm-worker1上面的client所在容器里面，再去ping下whoami `ping whoami`

![](https://images.notes.xuepincat.com/docker/swarm/47.png)

会看到还是可以 ping 通过，地址还会 `10.0.1.120`

此时有个疑问，有两个 whoami, 但是ping的地址到底ping的谁呢？其实这个 `10.0.1.120` 地址并不是任何一个service，任何一个whoami容器的地址，它其实是一个 `VIP`，是一个虚拟IP

可以通过 `nslookup` 命令查看

![](https://images.notes.xuepincat.com/docker/swarm/48.png)

这个命令是去DNS服务器上查询DNS NAME的

比如我在本机上输入下面命令

```
nslookup notes.xuepincat.com
```

![](https://images.notes.xuepincat.com/docker/swarm/49.png)

它会返回我博客的 DNS Name 和背后所对应的 IP 地址

再到swarm-worker1上面的client所在容器里面，使用 `nslookup whoami` 命令查看，会看到返回了IP地址，这个IP既不是swarm-worker2上whoami的IP，也不是swarm-manager上面whoami的IP,可以分别到两个节点上看下它们的IP

swarm-worker2:

![](https://images.notes.xuepincat.com/docker/swarm/50.png)

并没有看到 `10.0.1.120` 的地址 

swarm-manager:

![](https://images.notes.xuepincat.com/docker/swarm/51.png)

也没有看到 `10.0.1.120` 的地址 

会到 client 所在的 swarm-worker1 机器，使用 `nslookup tasks.whoami` 命令，会看到返回两组IP地址，这两组IP地址才是两个whoami所对应的容器的地址

`10.0.1.120` 虚拟IP和实际容器的IP有个map关系，通过map关系就可以找到运行的service的实际的地址

现在继续在 client 所在的机子上使用 `wget whoami:8000` 命令获取页面内容，然后再查看index.html 返回的内容

![](http://images.notes.xuepincat.com/docker/swarm/52.png)

会看到返回了whoami的host name

现在将index.html删掉，再使用 `wget whoami:8000` 命令获取下页面

![](https://images.notes.xuepincat.com/docker/swarm/53.png)

会看到两次返回的host name并不一样

因为现在一个有两个 whoami,去访问whoami的时候响应的whoami是不一样的，也就是这里面有个负载均衡，也就是 `10.0.1.120` 的地址背后实际上是几个实际的IP地址所对应的web服务器，每次访问 whoami 的时候都会去做负载均衡，比如第一次是第一个whoami做出的响应，第二次情况就变成了另外一个whoami作出的响应，这个负载均衡都是通过一个叫 `LVS` 实现的

* Routing Mesh 的两种体现

- Internal 

    Container 和 Container 之间的访问通过 overlay 网络实现 (通过VIP虚拟IP)

    讲的是在swarm里面连接到同一个overlay网络的容器它们之间是如何通信的，首先它们之间是通过 service name 通信的，service name 所对应的IP地址并不是容器的具体IP地址，而是一个虚拟网络，并且如果service做了横向扩展，那么通过VIP去访问的时候会去做负载均衡

- Ingress 

    如果服务有绑定接口，则此服务可以通过任意swarm节点的相应接口访问

    比如上面案例中绑定了8000端口，之前的wordpress绑定了80端口，虽然service 容器可能在swarm cluster里面的部分节点上运行，但是就可以在swarm的任意节点上通过对应的端口进行访问

* Internal Load Balancing

![](https://images.notes.xuepincat.com/docker/swarm/54.png)

如果有两个web service 和 一个 client,它们都连接到同一个 overlay 的网络，它们分别位于不同的节点上面，这时候client去访问web的时候是通过VIP访问的，VIP会自动将访问的会 Load Balancing(负载均衡)到每个service节点上，这个就是 Internal Load Balancing

* DNS + VIP + iptables +LVS

![](https://images.notes.xuepincat.com/docker/swarm/55.png)

比如在client的container里面访问另外的service的时候首先会去DNS中找到service name,将 service name解析成具体的VIP，当用户访问对应的8000端口，就会通过iptables以及IPVS去做负载均衡到不同的service所在的节点上

部署到swarm cluster里面的不同的service,它们连接到同一个overlay的网络上，它们之间的通信主要是有两块内容，service底层是容器，容器和容器的通信主要是通过overlay的网络，通知VXLAN Tunnel实现的；service和service之间的通信是通过VIP实现的，比如client的service要去访问web1的service，一个web可能有很多扩展，所以client在访问web的时候是访问web的VIP，VIP映射到具体的web的ContainerIP上，是通过LVS实现的

* Ingress Network

比如把一个wordpress服务部署到swarm cluster里面，可能会去做一个scale横向扩展，比如某一个容器会部署到cluster的某一个节点上，另外web会暴露一个端口比如8080，当去访问这个web service的时候会发现在swarm cluster的任意一个节点上访问8080都可以访问到这个wordpress,这个就是 Ingress Network 在起作用

![](https://images.notes.xuepincat.com/docker/swarm/56.png)

Ingress Network 的作用就是当去任何一个节点访问端口服务的时候，它会将服务通过本机连接的IPVS(Viutual Service) 给 Load Balancing 到真正具有service的swarm节点上,比如上图中访问Docker host3的8080，但是Docker host3上面并没有service,但是它会通过IPVS可以将请求转发到另外两台具有service的Docker host节点上,然后将response返回给我们

1. 在 swarm-manager 节点上查看下 whoami 的运行情况

```
docker service ps whoami
```

![](https://images.notes.xuepincat.com/docker/swarm/57.png)

它运行在了swarm-manager和swarm-worker2节点上

2. 在 swarm-manager 节点上通过 `curl 127.0.0.1:8000` 进行测试访问

![](https://images.notes.xuepincat.com/docker/swarm/58.png)

4. 在 swarm-manager 节点上再次通过 `curl 127.0.0.1:8000` 进行测试访问

![](https://images.notes.xuepincat.com/docker/swarm/59.png)

一直重复这样的操作它会做一个负载均衡

5. 到 swarm-worker1 的节点上使用 `curl 127.0.0.1:8000` ，因为 worker1 的节点上并没有 whoami 的 service

![](https://images.notes.xuepincat.com/docker/swarm/60.png)

会看到还是可以访问，这就是 Ingress Network 在起作用，这是怎么实现的呢？

6. 在worker1上输入下面命令查看本地转发规则

```
sudo iptables -nL -t nat
```

![](https://images.notes.xuepincat.com/docker/swarm/61.png)

其中与一段

```
Chain DOCKER-INGRESS (2 references)
target     prot opt source               destination
DNAT       tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:8000 to:172.18.0.2:8000
DNAT       tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:80 to:172.18.0.2:80
RETURN     all  --  0.0.0.0/0            0.0.0.0/0
```

它的转发规则是如果访问TCP，端口是8000端口的，也就是 `curl 127.0.0.1:8000`,它就会转发到 `172.18.0.2:8000`

先看下本地的IP和端口

![](http://images.notes.xuepincat.com/docker/swarm/62.png)

会看到有个叫 `docker_gwbridge` ，它的地址是 `172.18.0.1`,这个地址和上面转发的地址 `172.18.0.2` 是在同一个网段，所以可以猜测到 `172.18.0.2` 这个地址肯定是 docker_gwbridge 相连的一个网络

通过 `brctl show` 命令查看下

![](http://images.notes.xuepincat.com/docker/swarm/63.png)

会看到 docker_gwbridge 上有一些interfaces

接着通过 `docker network ls` 和 `docker network inspect docker_gwbridge` 查看下网络

![](https://images.notes.xuepincat.com/docker/swarm/64.png)

![](https://images.notes.xuepincat.com/docker/swarm/65.png)

会看到和 docker_gwbridge 相连的容器有三个,`curl 127.0.0.1:8000`,会转发到 `172.18.0.2:8000`，也就是 `ingress-sbox`,它是一个namespace,也就是说将数据包转发到了这个 namespace 里面去了

7. 进入 ingress-sbox namespace

```
sudo ls /var/run/docker/netns
```

```
sudo nsenter --net=/var/run/docker/netns/ingress_sbox
```

```
ip a
```

![](https://images.notes.xuepincat.com/docker/swarm/66.png)

当前已经处理 ingress-sbox 的 namespace 里面

![](https://images.notes.xuepincat.com/docker/swarm/67.png)

访问的是本地的 `127.0.0.1:8000`, 数据包通过 docker_gwbridge 转发到 ingress-sbox,ingress-sbox 其实是一个 namespace

然后通过下面命令查看下转发规则

```
iptables -nL -t mangle
```

![](http://images.notes.xuepincat.com/docker/swarm/68.png)

会看到这样一组数据

```
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination
MARK       tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:80 MARK set 0x106
MARK       tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:8000 MARK set 0x10d
```

它向8000端口做了一个 MARK set 0x10d,这个其实就是做负载均衡的，也就是通过LVS把向8000端口的数据包做 Load Balancing,将它转发到具体的实际的IP地址，这个实际的IP地址其实就是某一个service的地址

8. 安装 ipvsadm

退出 namespace,安装 ipvsadm

```
sudo yum install -y ipvsadm
```

安装完之后再进入到 ingress-sbox 中

```
sudo nsenter --net=/var/run/docker/netns/ingress_sbox
```

运行 `ipvsadm -l` 命令

![](https://images.notes.xuepincat.com/docker/swarm/69.png)

会看到有连个地址

```
FWM  269 rr
  -> 10.0.0.14:0                  Masq    1      0          0
  -> 10.0.0.15:0                  Masq    1      0          0
```

上面的两个地址就是 whoami 两个service container的地址，可以使用 `docker ps` 分别到 swarm-manager 和 swarm-worker 两个节点上看下它们的IP

所以说数据包在 ingress_sbox 通过 LVS 做了负载均衡，如果访问8000端口就会把数据转发到 `10.0.0.14` 和 `10.0.0.15` 做一个负载均衡

数据在进入 ingress_sbox 之后通过 iptables 和 IPVS就可以将数据正确的转发到相应的另外一个host中，它们的网络是 overlay 的网络，另外一个host中数据再进入到 Container-sbox 中
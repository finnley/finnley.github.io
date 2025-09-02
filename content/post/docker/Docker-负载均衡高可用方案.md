+++
title = 'title: 负载均衡高可用方案'
date = 2018-09-24T01:39:37+08:00
draft = true
categories = [ "Docker" ]
tags = [ "docker" ]
+++

前面利用Haproxy成功负载均衡的数据库集群，但是Haproxy是单节点部署的，如果这个节点出现了宕机，那么负载均衡方案就无效了。此时就需要进行冗余设计。

## 为什么要采用双机热备

* 单节点的Haproxy不具备高可用，必须要有冗余设计

![](/images/docker/78.png)

在PXC集群中，使用Haproxy做负载均衡，应用程序发过来的请求由Haproxy请求转发给具体的数据库实例，确实可以降低每一个数据库实例的负载，正常情况下这是没有问题。

但是一旦Haproxy出现了故障，应用程序就无法把数据的请求发送到Haproxy上了，那数据库的负载均衡也就失效了。所以Haproxy不能成为我们的瓶颈，Haproxy一定要设计成双节点或者是多节点的方案，一个节点挂掉后还有其他节点的Haproxy可以使用。

## 虚拟IP（Vrtual IP Address）

Haproxy的双机热备方案离不开一个技术，这个技术叫做 `虚拟IP`

* 是一种不与特定计算机或者特定计算机网卡相对应的IP地址。所有发往这个IP地址的数据包最后都会经过真实的网卡到达目的主机的目的进程。

### 虚拟IP的用处

虚拟IP主要是用来 `网络地址转换`，`网络容错` 和 `可移动性`。

虚拟IP比较常见的一个用例就是在 `系统高可用性（High Availability HA）` 方面的应用，通常如果系统出现宕机，为了提高系统对外服务的高可用性，就会采用主备模式进行高可用性的配置。当提供服务的主机M宕机后，服务会切换到备用主机S继续对外提供服务。而这一切用户是感觉不到的，在这种情况下系统对客户端提供服务的IP地址就会是一个虚拟IP，当主机M宕机后，虚拟IP便会漂浮到备机上，继续提供服务。

在这种情况下，虚拟IP就不是与特定计算主机或者特定某个物理网卡对应的了，而是一种虚拟或者是说逻辑的概念，它是可以自由移动自由漂浮的，这样一来既对外屏蔽了系统内部的细节，又为系统内部的可维护性和扩展性提供了方便。

## 利用Keepalived实现双机热备

### 双击热备细节

![](/images/docker/80.png)

首先要定义出虚拟IP，这里使用的双机热备，所以需要准备两个Haproxy，也就是说要在docker虚拟机上要启动两个容器，这两个容器各自运行Haproxy，Haproxy所在的容器还需要再安装一个程序，这个程序就叫做 `Keepalived`, `Keepalived` 是用来抢占虚拟IP的，所以我们在各自的Haproxy容器中安装好 `Keepalived` 之后，Keepalived就会抢占虚拟IP，抢到虚拟IP之后，另外一个没有抢到，那么它就会处在一个等待的状态。

抢到虚拟IP之后的 `Keepalived` 所在的容器叫作 `主服务器`，没有抢占到虚拟IP的 `Keepalived` 所在的容器叫做 `备用服务器`，两个 `Keepalived` 之间是有心跳检测的，如果 `备用服务器` 发现发送给 `主服务器` 的 `Keepalived` 心跳检测没有响应，也就是说 `主服务器` 可能出现了故障，这个时候 `备份服务器` 上的 `Keepalibed` 就有权将虚拟IP抢到手，这样就可以通过应用程序向虚拟IP发送数据库请求，我们不去关心虚拟IP对应哪一个Haproxy，因为一个Haproxy容器挂掉还有另外一个Haproxy容器来接替工作，这个就是双机热备的具体细节

### Haproxy双机热备方案

![](/images/docker/81.png)

* Docker内的虚拟IP不能为外网使用，所以需要借助宿主机Keepalived映射成外网可以访问的虚拟IP

右侧是数据库的集群，使用双机热备方案需要创建两个容器，分别部署Haproxy，里面安装好Keepalived,这样任何一个容器挂掉之后还有另外一个容器可以使用，这就是双机热备的冗余设计；然后两个Keepalived要抢占一个虚拟IP，这个虚拟IP是 `172.18.0.15`，从这个网段可以看出这个网段只能在docker内部使用，如果在局域网上想访问这个docker内部的虚拟IP怎么办呢？

我们需要在宿主机上安装Keepalived，让宿主机的Keepalived把某一个IP映射到docker的虚拟IP上。

我们安装好了Keepalived之后，通过命令行创建一个虚拟IP，这个虚拟IP假设是 `192.168.99.65`，那么将来应用程序向这个虚拟IP(192.168.99.65)发送数据请求，这个请求就会路由到docker的虚拟IP(172.18.0.15)上，这个虚拟IP(172.18.0.15)因为被某一个keepalived抢占，所以这个IP接收到的所有请求就会转发到对应的Haproxy上了，Haproxy再通过负载均衡技术把这个请求发送给某一个PXC的数据库节点，这就是总体的双击热备架构设计。

### 安装Keepalived

#### 指令

* Keepalived必须要安装在Haproxy所在的容器之内

```
apt-get update
apt-get install keepalived
```

#### 实操

* 进入Haproxy容器

```
docker exec -it h1 bash
```

![](/images/docker/82.png)

* 退出容器,只是退出交互界面，而不是停止容器的运行，可以使用 `exit` 命令

* 更新apt-get

```
apt-get update
```

![](/images/docker/83.png)

* 安装keepalived

```
apt-get install keepalived
```

### Keepalived配置文件

* Keepalived的配置文件是 `/etc/keepalived/keepalived.conf`

keepalived运行的时候要占抢虚拟IP，这个虚拟IP就要写在配置文件里。如果想在容器终端窗口通过vim编写配置文件，Haproxy镜像里面是没有vim的，所以需要在容器里面安装vim编辑器。

```
apt-get install vim
vim /etc/keepalived/keepalived.conf
```

* Keepalived配置文件

```
vrrp_instance VI_1 {
    state MASTER #Keepalived身份，有两个值一个是MASTER,一个是SLAVE（MASTER主服务，BACKUP备份服务。主服务要抢占虚拟IP，备用服务器不会抢占IP）
    interface eth0　#网卡设备
    virtual_router_id 51 #虚拟路由标识，MASTER和BACKUP的虚拟路由标识必须一致。标识可以是0-255
    priority 100 #MASTER权重要高于BACKUP 数字越大优先级越高
    advert_int 1 #MASTER和BACKUP节点间 同步检查的时间间隔，单位为秒。主备之间必须一致
    authentication { #主从服务器验证方式。主备必须使用相同的密码才能正常通信
        auth_type PASS
        auth_pass 123456
    }
    virtual_ipaddress { #虚拟IP地址。可以设置多个IP地址，每行一个
       	172.18.0.201
    }
}
```

* state MASTER: Keepalived这个节点的身份，有两个值一个是MASTER,一个是SLAVE。MASTER身份表示Keepalived启动之后是要争抢虚拟IP的，SLAVE身份表示启动keepalived之后不会去争抢虚拟IP，如果将每个keepalived配置文件都定义成 MASTER 身份，这些节点启动之后都会去争抢虚拟IP，只有一个节点抢到，其他keepalived节点身份自动降级为SLAVE；
* interface eth0: interface 规定的是网卡，也就是要定义虚拟IP，这个虚拟IP保存到什么网卡里面，这里是网卡的名字，eth0 这个网卡是docker虚拟机的网卡，网卡在局域网上是看不见的，所以把虚拟IP写入到 docker 的网卡里面，在宿主机是可以访问这个网卡，但是出了宿主机，在局域网其他地方是看不到这个eth0网卡的，所以需要在宿主机上将 eth0 这个网卡的虚拟IP映射到局域网上的某个虚拟IP上，所以后面还会在宿主机上安装 keepalived；
* virtual_router_id 51: 给 keepalived 起一个id，数字0-255之间随便选一个；
* priority 100: 抢占虚拟IP如果硬件配置较高的主机上的keepalived可以将权重设置高点，就可能优先抢占到虚拟IP，根据硬件配置调整权重数字，数字越大越能优先抢到虚拟IP；
* advert_int 1: keepalived之间有心跳检测，这个就是心跳检测的时间间隔，1表示1秒的意思；
* authentication: 心跳检测是要登录到某个keeplaived节点里面的，所以心跳检测需要开放的账号和密码；
* virtual_ipaddress: eth0里面定义的虚拟IP，比如这个数据库集群网段是 `172.18.0.*`,那么这个虚拟IP这边就定义的 `172.18.0.201`，这个就是 eth0 里面定义的虚拟IP，这个虚拟IP在docker内部是可以看到的，出了docker是看不到的；


在容器中输入下面指令

```
vim /etc/keepalived/keepalived.conf
```

* 我的keepalived.conf配置文件内容

```
vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 101
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 123456
    }
    virtual_ipaddress {
       	172.18.0.201
    }
}

```

### 启动Keepalived

* 启动Keepalived

```
service keepalived start
```


![](/images/docker/84.png)

* 启动之后在宿主机里可以 ping 通虚拟IP，如果能ping通就表示虚拟IP配置成功了

```
ping 172.18.0.201
```

## 创建第二个haproxy （h2）

* 创建haproxy容器后台

```
docker run -it -d -p 4003:8888 -p 4004:3306 -v /home/soft/haproxy:/usr/local/etc/haproxy --name h2 --privileged --net=net1 --ip 172.18.0.8 haproxy
```

* 进入容器后台

```
docker exec -it h2 bash
```

* 启动haproxy -f表示加载配置文件

```
haproxy -f /usr/local/etc/haproxy/haproxy.cfg
```

这个时候可以在浏览器中输入 `http://IP:4003/dbs` 进行查看，数据库连接工具可以创建H2节点，端口是4004


* h2安装keepalived

```
apt-get update
apt-get install keepalived
apt-get install vim
vim /etc/keepalived/keepalived.conf
```

* 配置文件

```
vrrp_instance  VI_1 {
    state  MASTER
    interface  eth0
    virtual_router_id  51
    priority  100
    advert_int  1
    authentication {
        auth_type  PASS
        auth_pass  123456
    }
    virtual_ipaddress {
        172.18.0.201
    }
}
```

* 启动h2的keepalived

```
service keepalived start
```

然后重新打开一个连接

```
ping 172.18.0.201
```

## 宿主机上安装keepalived

```
sudo yum install -y keepalived
```

配置文件信息 宿主机上的配置文件位置为 `/etc/keepalived/keepalived.conf`

修改配置信息如下：

```
vrrp_instance VI_1 {
    state MASTER
    interface eth1
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.1.156
    }
}

virtual_server 192.168.1.156 8888 {
    delay_loop 3
    lb_algo rr 
    lb_kind NAT
    persistence_timeout 50
    protocol TCP
    
    real_server 172.18.0.201 8888 {
        weight 1
    }
}

virtual_server 192.168.1.156 3306 {
    delay_loop 3
    lb_algo rr 
    lb_kind NAT
    persistence_timeout 50
    protocol TCP
    
    real_server 172.18.0.201 3306 {
        weight 1
    }
}
```

第一段是在宿主机 `eth1` 的网卡里定义一个虚拟IP，因为之前的 dcoker 虚拟IP在局域网上无法直接访问，所以需要把 docker 的虚拟IP映射到局域网上的虚拟IP，所以先定义一个局域网的虚拟IP。定义的虚拟IP的地址是 `192.168.1.156`，这个虚拟IP是保存带 `eth1` 这个网卡上的。

Docker虚拟机里面的网卡名字叫做eth0，eth0这个网卡的名字，必须要进入docker容器才能看到，在容器之外看不到eth0这个网卡。我本地路由器的IP网段是1，所以不要按照我这个网段写虚拟IP，根据你本地实际的网段来填写虚拟IP地址。比如我使用vagrant+virtualbox创建的宿主机IP为 `192.168.44.44`,网段是 `44` 的，那么我可以定义个虚拟IP也是 `44` 的，比如 `192.168.44.150` 。

定义完虚拟IP之后还需要为这个虚拟IP设置一些转发端口。因为 docker 内的虚拟IP在局域网上无法直接访问，应用程序的请求是发给局域网的虚拟IP的，局域网的虚拟IP会把请求转发到 docker 的虚拟IP上，转发规则就是上面配置的第二段，`192.168.1.156` 这个是宿主机的虚拟IP，开放的端口是 `8888`，`172.18.0.201` 这个就是要转发的 docker 虚拟IP，请求从 `192.168.1.156` 进来，然后转发到 `172.18.0.201` 这个IP上，这样局域网上的其他主机就可以访问 docker 容器提供的 haproxy 负载均衡了。

docker 虚拟IP的端口是 `8888` 端口，这个 `8888` 不是 haproxy 图形管理界面的端口，这个地方也不应该是 `4001` 端口， `4001` 是宿主机的端口，`172.18.0.201` 这个虚拟IP对应的某个 keepalived 运行的容器，这个容器里面的 haproxy 后台管理端口就是 `8888`。

第三段就是数据库转发规则，`192.168.1.156` 是宿主机上的虚拟IP地址，开放的转发端口是 `3306`，`3306` 端口接收到请求后转发到 docker 的虚拟IP，这个 `201` 虚拟IP会被某个 keepalived 抢占，所以向 `201` 发送的数据库请求，keepalived 所在容器内的 haproxy 就能接收到请求。

* 宿主机上keepalived的配置文件，上传到服务器,宿主机上文件位置为 `/etc/keepalived/keepalived.conf`，然后启动宿主机keepalived

```
sudo service keepalived start
```

![](/images/docker/85.png)

* 然后ping一下文件中设置的IP

```
ping 192.168.1.156
```

* 打开浏览器输入网址,地址是配置的虚拟IP `192.168.1.156`,端口8888,提示输入用户名和密码，用户名为admin,密码是abc123456

```
192.168.1.156:8888/dbs
```

![](/images/docker/86.png)

在上图中我们访问的局域网内的虚拟IP，这个请求就被转发到Docker里面的虚拟IP，Dockers的虚拟IP指不定被哪一个容器中h1或者h2中的keepalived抢占了，抢占完经由haproxy处理，然后上图看到的画面就是某一个容器内的haproxy的监控。

> 注意：在最后宿主机上配置keepalived的时候如果输入的网址不成功可能是因为防火墙的原因，我这里将防火墙关掉就可以了。

```
systemctl stop firewalld
```

* 现在打开数据库管理工具，地址为局域网虚拟IP，端口是3306，密码是 `abc123456`

![](/images/docker/87.png)

测试连接成功，这次发起的数据库请求，因为采用的是双机热备方案，这次请求不一定经由哪一个haproxy发到数据库节点上，然后就是通过这个局域网的虚拟IP来做数据库的增删改查也是可以的，比如

![](/images/docker/88.png)

* 我向表中添加一条数据，这次insert请求也是不一定经由哪一个haproxy路由到数据库节点上，现在查看比如DB1节点

![](/images/docker/89.png)

* 再打开DB5节点

![](/images/docker/90.png)

接下来测试双机热备的高可用性，使用双机热备方案就是为了避免一个haproxy节点挂掉之后数据库的负载均衡不能使用，但是现在有两个节点，我挂掉任何一个节点还有另外一个节点依然可以使用，当前我启动了两个容器h1和h2。

* 现在我打算将h1容器停掉

```
docker pause h1
```

![](/images/docker/91.png)

这样h1容器里面haproxy就用不了了，但是现在因为双机热备方案还有h2容器来承担数据库的负载均衡，现在在数据库的客户端再试验一下，我再次添加一条记录。

![](/images/docker/92.png)

* 现在打开几个数据库节点看一下

![](/images/docker/93.png)

![](/images/docker/94.png)

![](/images/docker/95.png)

![](/images/docker/96.png)

![](/images/docker/97.png)

从图中可以看到双机热备中挂掉一个节点还有另外一个节点可以正常运行。

* 先将h1节点恢复

```
docker unpause h1
```

* 如果想要彻底停止h1容器

```
docker stop h1

```
* 如果想要启动h1

```
docker start h1
```

* 但是这样容器里面的haproxy和keepalived并没有启动，所以还要进入到容器里面启动haproxy和keepalived两个服务，进入容器

```
docker exec -it h1 bash
```

* 启动服务

```
service haproxy start
service keepalived start
```

现在可能我将电脑关闭了，然后再打开电脑进入docker虚拟机启动pxc容器发现启动之后pxc容器闪退了，因为管理pxc集群方案比较复杂，如果挂掉pxc集群中的一些节点，将来将它们再启动的时候，它们的启动顺序非常重要。

## 暂停PXC集群方法

```
sudo vim /etc/sysctl.conf
#文件中添加 net.ipv4.ip_forward=1 这个配置
sudo systemctl restart network
```

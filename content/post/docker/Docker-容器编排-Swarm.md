+++
title = '容器编排 Swarm'
date = 2020-07-24T00:09:23+08:00
draft = true
categories = [ "Docker" ]
tags = [ "docker" ]
+++

## 背景

实际生产环境比较复杂，有可能部署在一台机器上根本无法满足实际需求，可能包含的容器比较多，在生产环境中遇到的情况比如在集群中部署应用，就会涉及到非常多的容器，从而也会给我们带来一些问题

- 怎么去管理这么多容器？
- 需要增加容器，怎么能方便的横向扩展？
- 如果容器 down 了，怎么能自动恢复？
- 如何去更新容器而不影响业务？
- 如何去监控追踪容易的健康状态？
- 怎么去调度容器，可能涉及到很多集群，可能这台机器资源比较紧张，另外机器资源比较充足？
- 如何保护隐私数据？

Swarm 是内置于 Docker 的编排工具，容器编排工具并不只有 Swarm

## 架构

![](https://images.notes.xuepincat.com/docker/swarm/1.png)

Swarm 是一个集群的架构，集群就肯定有节点，节点又有角色

Swarm 中节点有两种角色，一个角色叫做 `Manager`,另外一个角色叫做 `Worker`

Manager 角色是整个 Swarm 集群的大脑，为了避免单点故障，Manager 的节点一般至少两个，如果有多个聚会涉及到状态的同步，就是通过这个 Manager 所做的事情，产生的数据如果同步到另外的 Manager 节点上，这里又涉及到内置的分布式存储数据库，使用 `Raft` 协议去做了一个同步,Raft 能确保 Manager 节点它们之间的信息是对称的，同步的，不会出现脑裂的情况

Manager 的第二个角色是 `Worker`,它实际上就是工作的节点，大部分容器部署到 Cluster 都会运行在 Worker 节点上面，当然 Manager 节点也是可以运行的，但是主要还是 Worker，因为 Worker 节点比较多一点，Worker 节点之间也有些数据需要去同步，它们之间通过叫 `Gossip network` 的网络去做信息同步

* Services 和 Replicas

![](https://images.notes.xuepincat.com/docker/swarm/2.png)

一个 service 和 compose 里面的 service 类似，一个 service 就代表了一个容器，但是在 Replicas 模式下需要横向扩展，在部署的时候，一个 Replicas 就是一个容器

比如部署一个service 是 nginx，需要做3个扩展，实际上部署的时候回产生3个容器，这3个容器会通过调度系统将它们调度到不同的node上面，也就是通过左边的swarm manager节点部署service的时候是不知道service最终是运行在哪些swarm节点上的，这个会根据swarm的调度算法去计算，比如看那些负载较轻，有充足的内存资源和CPU资源，然后就会将容器调度到这台机器上

* 服务创建与调度

![](https://images.notes.xuepincat.com/docker/swarm/3.png)

在swarm节点上做所有的决策，最中结果要把service部署到哪台node上面，然后再去执行实际的操作

## 创建一个三个节点的 Swarm 集群

* Vagrant + VirtualBox

Dockerfile

```
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.require_version ">= 1.6.0"

boxes = [
    {
        :name => "swarm-manager",
        :eth1 => "192.168.205.10",
        :mem => "1024",
        :cpu => "1"
    },
    {
        :name => "swarm-worker1",
        :eth1 => "192.168.205.11",
        :mem => "1024",
        :cpu => "1"
    },
    {
        :name => "swarm-worker2",
        :eth1 => "192.168.205.12",
        :mem => "1024",
        :cpu => "1"
    }
]

Vagrant.configure(2) do |config|

  config.vm.box = "centos/7"

  boxes.each do |opts|
      config.vm.define opts[:name] do |config|
        config.vm.hostname = opts[:name]
        config.vm.provider "vmware_fusion" do |v|
          v.vmx["memsize"] = opts[:mem]
          v.vmx["numvcpus"] = opts[:cpu]
        end

        config.vm.provider "virtualbox" do |v|
          v.customize ["modifyvm", :id, "--memory", opts[:mem]]
          v.customize ["modifyvm", :id, "--cpus", opts[:cpu]]
        end

        config.vm.network :private_network, ip: opts[:eth1]
      end
  end

  config.vm.synced_folder "./labs", "/home/vagrant/labs"
  config.vm.provision "shell", privileged: true, path: "./setup.sh"

end
```

1. 查看

```
vagrant status
```

![](https://images.notes.xuepincat.com/docker/swarm/4.png)

进入 swarm-manager 节点

```
vagrant ssh swarm-manager
```

可以通过下面命令查看下 swarm 的一些命令

```
docker swarm --help
```

![](https://images.notes.xuepincat.com/docker/swarm/5.png)

2. 初始化

swarm-manager 节点的IP是 `192.168.205.10`

首先在 swarm-manager 上运行，先运行的节点会成为 swarm cluster 的 manager 节点

```
docker swarm init --advertise-addr=192.168.205.10
```

```
Swarm initialized: current node (9aai900o1j1wq5wbw0sgfkyo7) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-3usjz2kim1pbziq9eplg5gyyg8y12d2u20zm4y9x5sq51f7mv8-bldu89yg8j9pa2zpmcshym8ys 192.168.205.10:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

![](https://images.notes.xuepincat.com/docker/swarm/6.png)

如上图，初始化完成，当前的节点就是 manager 节点

如果想要添加 worker 节点，只需要此基础上去运行 `swarm join ...`, 带上 token 和 swarm cluster 的IP地址和端口

将给出的信息拷贝

3. 进入 swarm-worker1 节点

```
vagrant ssh swarm-worker1
```

4. 运行上面拷贝的内容

```
docker swarm join --token SWMTKN-1-3usjz2kim1pbziq9eplg5gyyg8y12d2u20zm4y9x5sq51f7mv8-bldu89yg8j9pa2zpmcshym8ys 192.168.205.10:2377
```

![](http://images.notes.xuepincat.com/docker/swarm/7.png)

5. 进入到 swarm-manager 节点查看状态

```
docker node ls
```

![](https://images.notes.xuepincat.com/docker/swarm/8.png)

看到有两个节点，一个是 swarm-manager 节点，一个是 swarm-worker1 节点

6. 进入 swarm-worker2 运行上面的拷贝的内容

```
docker swarm join --token SWMTKN-1-3usjz2kim1pbziq9eplg5gyyg8y12d2u20zm4y9x5sq51f7mv8-bldu89yg8j9pa2zpmcshym8ys 192.168.205.10:2377
```

![](https://images.notes.xuepincat.com/docker/swarm/9.png)

7. 再到 swarm-manager 节点查看状态 

```
docker node ls
```

![](https://images.notes.xuepincat.com/docker/swarm/10.png)

已经有了三个节点，一个 Leader ,两个 Workder

## Swarm Service

1. 进入 swarm-manager 节点创建 service

```
docker service create --name demo busybox sh -c "while true;do sleep 3600;done"
```

可通过下面命令查看

```
docker service ls
```

![](https://images.notes.xuepincat.com/docker/swarm/11.png)

上面操作是通过 service 创建了一个 Container ，但是Container是在哪一台机器上呢？

可以通过下面命令查看service的详细信息，也就是container的详细信息

```
docker service ps demo
```

![](https://images.notes.xuepincat.com/docker/swarm/12.png)

从 `NODE` 一列可以看到运行在 swarm-manager 这台机器上

另外还可以通过 `docker ps` 查看下

![](https://images.notes.xuepincat.com/docker/swarm/13.png)

可以看到有个 UP 的container，名字并不是叫 `demo`, 而是叫 `demo.1.zj0bxp52e3xijwa05l2qhi1ct`,也就会说 `demo` 是 service 的名字，并不是service创建的容器的名字

使用 `docker service ls` 看到的信息中 `MODE` 为 `replicated`, `REPLICAS` 为 ``1/1,它们表示这个创建的service是可水平扩展的

2. 扩展

```
docker service scale demo=5 
```

可以通过 `docker service ls` 查看结果

![](https://images.notes.xuepincat.com/docker/swarm/14.png)

`REPLICAS` 变成了 `5/5`,分母上的数字表示 service 的 scale，比如 demo=5

使用命令查看详细信息

```
docker service os demo
```

![](https://images.notes.xuepincat.com/docker/swarm/15.png)

可以看到现在service有5个，而且这个5个容器是平均分布在cluster中的，有两个在manager上，有两个在worker2上，一个在worker1上

3. 进入worker1查看

```
docker ps
```

![](https://images.notes.xuepincat.com/docker/swarm/16.png)

会看到有一个 container

4. 进入worker2查看

![](https://images.notes.xuepincat.com/docker/swarm/17.png)

会看到有两个 container

上面就是通过 scale 将 service 横向扩展成5个后在cluster中的分布情况

5. 在 worker2中将一个容器强制删除

```
docker rm -f 202c7793c660
```

![](https://images.notes.xuepincat.com/docker/swarm/18.png)

然后再到 swarm-manager 上通过 `docker service ls` 查看下

![](https://images.notes.xuepincat.com/docker/swarm/19.png)

结果有最初的一瞬间是 `4/5`,然后过了会就变成了 `5/5`

通过 `docker service ps demp` 查看下

![](http://images.notes.xuepincat.com/docker/swarm/20.png)

可以看到在 worker2 上面有一个扩展是 Shutdown,但是在 worker1 上面又启动了一个，现在到 worker1 上面使用 `docker ps` 查看下，就会看到有两个 container

![](https://images.notes.xuepincat.com/docker/swarm/21.png)

从上面的过程可以看到 `scale` 不仅仅能做到横向扩展，而且能确保一定数目的扩展是有效的，也就是说当发现有部分节点service失效了或者是退出了或者是shutdown了，它会发觉，然后会通过在cluster上任意的节点上启动一个service去修补，从而达到特定数目的 scale,确保数据一定是特定的，并且是有效的

6. 删除 service

```
docker service rm demo
```

在 manager 节点行删除，如果在worker节点上删除会提示下面信息

![](https://images.notes.xuepincat.com/docker/swarm/22.png)

在manager节点上删除后查看信息就看不到了,在worker节点上通过 `docker ps` 也需要等待会才能消失，看不到

![](https://images.notes.xuepincat.com/docker/swarm/23.png)

`docker service rm service_name` 使用该命令删除时很快，但是后台却有一个很复杂的操作，它会看这个sevice一共有多少个容器在cluster里面，并且在哪些机子上面，然后需要将这些容器进行删除，这个过程可能比较慢，所以当执行完这个命令的时候再到其他机器上查看的时候可能还会看到，然后等待一段时间后就看不到了
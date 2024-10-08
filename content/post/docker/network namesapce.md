+++
title = '网络命名空间'
date = 2020-04-20T21:03:53+08:00
draft = true
categories = [ "Docker" ]
tags = [ "docker" ]
+++

## 什么是 Network Namespace

通过后台执行的方式 `docker run -d` 创建一个 Container, 并为该容器起一个名字为 test1, 使用的 Image 为 `busybox`, 它是一个非常小的 linux image, 在里面运行一个 while 无限循环的 shell 命令,这个 Container 会一直在后台执行

```
sudo docker run -d --name test1 busybox /bin/sh -c "while true; do sleep 3600; done"
```

![](/images/docker/network-namespace/1.png)

通过 `docker exec ...` 进入容器

```
sudo docker exec -it 7cde98cd0902 /bin/sh
```

![](/images/docker/network-namespace/2.png)

此时已经进入到了busybox这个Container里面，可以运行一些命令，如

```
ip a
```

该命令显示当前容器的网络接口,一个是 loopback 0,也就是本地回环口，地址是 127.0.0.1，还有一个是 eth0@if6，它有MAC地址，还有个127.17.0.2的IP地址

![](/images/docker/network-namespace/3.png)

图片的内容其实就是网络的 `Namespace` 

现在 `exit` 退出,当前还是处于 docker host 中，在这个里面也可以运行 `ip a`

![](/images/docker/network-namespace/4.png)

会看到 host 里面也有一些接口，这些接口和在 Container 里面看到的接口是不一样的，而且它们是完全隔离的

之前通过 `docker run ...` 创建了一个容器，同时也就创建了一个 linux 的 network namespace, 这个 network namespace 和主机的 network namespace 是隔离开的， 这就是通过 Linux Network Namespace 实现了容器网络的隔离

同理，如果再创建第二个容器，它也会有一套独立的 Network Namespace 

```
sudo docker run -d --name test2 busybox /bin/sh -c "while true; do sleep 3600; done"
```

![](/images/docker/network-namespace/5.png)

现在不进入 test2 的容器，而是直接使用 `ip a` 命令

```
sudo docker exec 95ebe7e56051 ip a
```

![](/images/docker/network-namespace/6.png)

此时会打印出 test2 这个容器的网络命名空间

![](/images/docker/network-namespace/213.png)

第一个容器的地址是 `172.17.0.2`, 第二个容器的地址是 `172.17.0.3`， 它们两个是一套独立的网络命名空间，而且在test1的容器里面是可以ping通test2的地址的

比如现在进入test1

```
sudo docker exec -it 7cde98cd0902 /bin/sh
```

ping test2 容器的 ip

```
ping 172.17.0.3
```

![](/images/docker/network-namespace/7.png)

所以刚才创建的两个容器它们的网络是相通的，也就说明它们两个独立的Namespace也是可以通信的

## Network Namespace 管理

* 创建

```
sudo ip netns add test1
```

* 查看本机的已有的 network namespace

```
sudo ip netns list
```

* 删除

```
sudo ip netns delete test1
```

![](/images/docker/network-namespace/8.png)

* docker创建的network namespace是隐藏了，没法通过ip命令查看

现在重新创建两个 network namespace

```
sudo ip netns add test1
sudo ip netns add test2
```

![](/images/docker/network-namespace/9.png)

* 查看 Linux 下 Namespace IP

通过 `docker run ...` 创建了两台 test1 和 test2 的容器，每个容器都有自己的独立的 Network Namespace, 可以通过 `docker exec ...` 查看 network namespace 里面的IP地址、端口
同理怎么查看通过Linux Namespace创建的IP呢？

下面查看通过linux创建的network namespace的ip

如查看test1

```
sudo ip netns exec test1 ip a
```

![](/images/docker/network-namespace/10.png)

在 test1 的 network namespace 里面执行 `ip a` 命令，会看到有一个本地的回环口，这个回环口的特点是它没有地址，它没有127.0.0.1的这个地址，另外它的状态是 `DOWN` 的，也就是它没有 `UP` 起来，没有运行起来

除了可以在命令空间里面运行 `ip a` 之外，还可以运行 `ip link`,就像直接在主机下运行 `ip link` 一样,比如下面的可以看到本机的一些link,并且像本机的一些回环口状态都是 `UP` 的，有些是 `DOWN` 的

![](/images/docker/network-namespace/11.png)

现在在 test1 里面执行 ip link,会看到只有一个 link， 

```
sudo ip netns exec test1 ip link
```

![](/images/docker/network-namespace/12.png)

* 启动

如果想将 ip loopback 0的端口给 UP 起来，执行下面命令

```
sudo ip netns exec test1 ip link set dev lo up
```

上面命令的意思表示在 test1 的 Namespace 执行了 `ip link set dev lo up` 这条命令，这条命令就是让 lo 0的端口 UP 起来，因为之前是 `DOWN` 的

![](/images/docker/network-namespace/13.png)

但是现在看到它的状态变成了 'UNKNOWN'，为什么没有启动起来呢？

看上面的一张图片，其实本地的回环端口也是 `UNKNOWN`, 端口如果要 UP 起来是需要两端是连起来的，比如 etho 会去和 Mac 里面虚拟化的一个端口连起来，单个端口是无法 UP 起来的，必须是一对

![](/images/docker/network-namespace/14.png)

比如本机有两个 linux network namespace,test1和test2,它们分别有一个本地的回环端口，现在将 test1 和 test2 连起来，连接起来就需要接口，类似的比如将两台计算机连起来需要网线一样，网线必须插到每个计算机的网口上，同样的在 linux network namespace 里面有个叫 `Veth pair` 的东西，我们可以通过创建一对 `Veth pair`,然后将其中一个放入到 test1 的 network namespace 里面，另一个放入到 test2 的network namespace 里面，这样的话这一对端口就通过了一条线连起来了，因为这两个端口分别是在两个 network namespace 里面，所以说如果给两个端口都配个ip地址，它们两个就可以通了，这也就是刚开始通过 busybox 创建了两个容器，然后运行起来，它们就可以相互ping通，这个原理其实是一样的

下面就是使用 Linux 的命令创建去创建一对接口，然后放到 namespace 里面，并为它们配IP地址，然后去ping

* 创建一对veth接口

```
sudo ip link add veth-test1 type veth peer name veth-test2
```

然后再去执行 `ip link`,会发现多了一对 veth 的接口，分别是 veth-test2 和 veth-test1,它们都有Mac地址，但是没有IP，并且状态是都 DOWN

![](/images/docker/network-namespace/15.png)

* 将 veth-test1 接口添加到 namespace test1 中

上面看到本机执行 ip link 可以看到 9和10两个接口，在 test1 的 namespace 里面 只有 loopback 0

```
sudo ip netns exec test1 ip link
```

![](/images/docker/network-namespace/16.png)

然后执行下面命令,将 veth-test1 接口添加到 namespace test1 中

```
sudo ip link set veth-test1 netns test1
```

执行完之后看下 test1 namespace 里面

```
sudo ip netns exec test1 ip link
```

![](/images/docker/network-namespace/17.png)

![](/images/docker/network-namespace/18.png)

从图片中可以看到 veth-test1 端口 已经添加到 test1 中了

接着再看下本地的 ip link,会发现10已经不见了，只剩下9了，说明已经把10添加到 test1里面去了

* 将 veth-test2 添加到 namespace test2 中

```
sudo ip link set veth-test2 netns test2
```

然后再去看下本地,发现9也不见了

![](/images/docker/network-namespace/19.png)

再看下test2的namespace里面也多了个端口

```
sudo ip netns exec test2 ip link
```

![](/images/docker/network-namespace/20.png)

上面的操作就是将两个 veth 分别添加到 linux 的test1 和 test2 的 namespace 中了，但是现在他们的状态都是 DOWN 的，而且没有IP地址，只有MAC地址

* 为两个端口配置IP地址

```
sudo ip netns exec test1 ip addr add 192.168.1.1/24 dev veth-test1
```

```
sudo ip netns exec test2 ip addr add 192.168.1.2/24 dev veth-test2
```

比如第一条命令表示在 test1 的 namesapce 里面去执行后面的 `ip addr add ...` 命令，也就是给 dev veth-test1 端口分配一个 192.168.1.1 的IP地址，掩码是24

同理test2也是

然后再查看下 test1 和 test2 的ip link 情况

```
sudo ip netns exec test1 ip link
```

```
sudo ip netns exec test2 ip link
```

![](/images/docker/network-namespace/21.png)

此时去查看的时候还是没有看到 IP 地址的，而且还是 `DOWN` 的

* 启动两个端口

在test1 的namespace 里面将 veth-test1的端口启动起来，同理在test2的namespace里面将 veth-test2的端口也启动起来

```
sudo ip netns exec test1 ip link set dev veth-test1 up
```

```
sudo ip netns exec test2 ip link set dev veth-test2 up
```

现在再去查看下 test1 和 test2 的ip link 情况

test1

```
sudo ip netns exec test1 ip link
```

```
sudo ip netns exec test1 ip a
```

![](/images/docker/network-namespace/22.png)


![](/images/docker/network-namespace/23.png)

同样看下 test2 的地址，地址也有了，状态也是 UP 了

![](/images/docker/network-namespace/24.png)

  上面的一些操作分别给veth-test1端口去配IP地址，然后 veth-test2 也配IP地址，并且将它们UP起来，现在也就是 test1和test2 namespace已经连接起来了，连起来后如果在test1里面test2接口的IP地址是什么情况呢？

* ping 联通测试

test1 现在本地的地址是 192.168.1.1

```
sudo ip netns exec test1 ip a
```

然后去ping test2,发现已经通了

```
sudo ip netns exec test1 ping 192.168.1.2
```

![](/images/docker/network-namespace/25.png)

同样的在test2中去ping test1的IP

```
sudo ip netns exec test2 ping 192.168.1.1
```

![](/images/docker/network-namespace/26.png)

上面就是通过 Linux 的 Network Namespace 做的一个实验，这个和刚开始通过 busybox 创建的两个容器的之间是通的原理是基本上是完全一样的,如果创建一个容器，同时也会创建一个 network namespace, 就是这个容器的独立的网络命令空间，因为容器它本身会有自己的IP地址

比如通过 `docker run ...` 基于 busybox 创建了两个 Container, 一个test1,一个test2,然后通过 exec 执行 `ip a` 命令，在 test1 里面看到地址是 172.17.0.2,在test2 里面看到的地址是 172.17.0.3

并且在 test1 (172.17.0.2)里面是可以ping通test2的 (172.17.0.3)

```
sudo docker exec -it test1 /bin/sh
ping 127.0.0.3
```

![](/images/docker/network-namespace/27.png)

这种就是两个 network namespace 是连在一起的，上面的实验是通过一对 veth 接口连在一起的，在 docker 中两个container又是怎么连在一起的呢？另外在容器里面也是可以访问外部网络的比如访问百度，这些又是怎么实现的呢？
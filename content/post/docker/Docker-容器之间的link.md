+++
title = '容器之间的link'
date = 2020-04-27T22:58:24+08:00
draft = true
categories = [ "Docker" ]
tags = [ "docker" ]
+++

## 场景

比如有两个容器，一个容器启的是数据库服务，另一个容器启的后台服务，这个后台服务需要访问数据库，就需要知道另一个数据库容器的IP地址，也就是首先需要创建一个MySQL的数据库的容器，然后通过 docker 命令可以查出这个容器的IP地址,比如是172.17.0.3,然后再去启动后台容器服务的时候将 172.17.0.3 这个地址传入到参数中作为数据库的参数，这样后台服务才能连接到数据库的容器里面

这种情况会出现一个问题，就是我们写代码的时候不知道将来数据库容器的IP地址是什么

但是我们可以通过一种方式来给数据库容器取一个名字，我们就可以通过这个名字来访问它，就不需要知道IP地址了

就是在创建容器之前，将名称定好，名称是不会变的，就可以通过名称去访问容器，这个场景就可以通过docker中一个 `Link` 的机制，可以在创建第二个容器的时候，可以将它link到第一个容器上，这样去访问第一个容器的时候，就可以通过name访问了

## Link

现在有个test1的容器

![](https://images.notes.xuepincat.com/docker/link/1.png)

然后再去创建 test2 容器,在原先创建test2容器的基础上添加 `--link test1`,也就是将 test2 link 到 test1 上面

```
sudo docker run -d --name test2 --link test1 busybox /bin/sh -c "while true; do sleep 3600; done"
```

![](https://images.notes.xuepincat.com/docker/link/2.png)

先查看下test1的IP地址

```
sudo docker exec test1 ip a
```

test1 的IP地址是 172.17.0.2

接着进入刚创建的 test2 容器

```
sudo docker exec -it test2 /bin/sh
```

![](https://images.notes.xuepincat.com/docker/link/3.png)

ping 下 test1 的IP

```
ping 172.17.0.2
```

![](https://images.notes.xuepincat.com/docker/link/4.png)

是可以 ping 通的

但是去ping 下 test1的名字

![](https://images.notes.xuepincat.com/docker/link/5.png)

发现也是可以 ping 通的，这个为什么呢？

这就是因为在创建test2容器的时候添加了 `--link test1` 参数,相当于给test2添加了dns记录，对于test2来讲，将test1传进来，test1是已经建立好的，就可以知道test1的IP地址是什么了，也就出现了在test2的容器里面不需要知道test1的具体IP地址，只需要知道它的名字，而这个名字可以提前去指定，这样的话在test2里面假如有个数据库MySQL服务，就可以通过 `name:port` (test1:3306)去访问test1的数据库了

`exit` 退出 test2,进入 test1 容器,test2 的地址是 172.17.0.3

```
sudo docker exec -it test1 /bin/sh
```

ping test2 的地址肯定是可以的

```
ping 172.17.0.3
```

![](https://images.notes.xuepincat.com/docker/link/6.png)

如果去 ping test2 的名字呢？

```
ping test2
```

![](https://images.notes.xuepincat.com/docker/link/7.png)

发现ping不通，这是因为 link 有个类似方向的概念，上面的命令式 test2 link到test1,在test2中可以ping通test1,并不是test1 link 到 test2,所以只能在test2中ping test1

上面废话这么多，link 实际中用的并不多

停止 test2 并删除,然后重新创建 test2, 没有添加 link

```
sudo docker stop test2
```

```
sudo docker rm test2
```

```
sudo docker run -d --name test2 busybox /bin/sh -c "while true; do sleep 3600; done"
```

现在有两个容器 `sudo docker ps`

![](https://images.notes.xuepincat.com/docker/link/8.png)

输入下面命令查看网络情况

```
sudo docker network ls
```

![](https://images.notes.xuepincat.com/docker/link/9.png)

会看到有一个 bridge, 如果默认创建容器的时候会默认连bridge, 但是实际上在创建容器的时候可以指定 network,可以指定不使用bridge, 可以使用 host 或者是 none, 甚至可以自己去创建 bridge, 下面就操作下如何创建一个 bridge,然后将容器连接到创建的bridge上

## 创建 network

```
sudo docker network create
```

![](https://images.notes.xuepincat.com/docker/link/10.png)

可以添加参数 `-d bridge`, 比如

```
sudo docker network create -d bridge my-bridge
```

![](https://images.notes.xuepincat.com/docker/link/11.png)

这样就创建了一个网络，可以查看下

```
sudo docker network ls
```

![](https://images.notes.xuepincat.com/docker/link/12.png)

图中可以看到一个NAME是my-bridge,DRIVER是bridge的网络，这个和docker0类似

输入 `brctl show` 可以显示本地的Linux Bridge

```
brctl show
```

![](https://images.notes.xuepincat.com/docker/link/13.png)

会发现多了一个 `br-40f36f9190b5`，这个也就刚创建的 my-bridge

现在已经有了一个新的network,在创建容器的时候可以通过 `--network` 指定要连接的网络，比如现在新建一个 test3 的容器,如果直接执行，它会默认连接 docker0 的网络

## 指定 network

```
sudo docker run -d --name test3 --network my-bridge busybox /bin/sh -c "while true; do sleep 3600; done"
```

![](https://images.notes.xuepincat.com/docker/link/14.png)

现在创建的test3 也就连接到 my-bridge 上了

通过 `brctl show` 查看一下

![](https://images.notes.xuepincat.com/docker/link/15.png)

会看到新的 bridge 已经有接口了，之前还是没有的

当然也可以通过下面命令查看下

```
sudo docker network inspect 40f36f9190b5
```

![](https://images.notes.xuepincat.com/docker/link/16.png)

并且在Container中可以看到test3,这个test3是连在我们自己创建的my-bridge上面，它的地址是 172.18.0.2

上面就是新建容器并指定网络

对于之前已经新建的容器 test1 和 test2 来讲，其实也是可以link到新建的my-bridge上面,可以通过下面命令

```
sudo docker network connect my-bridge test2
```

执行完再去看下my-bridge

```
sudo docker network inspect 40f36f9190b5
```

![](https://images.notes.xuepincat.com/docker/link/17.png)

可以看到 test2 也已经连接到 my-bridge 上了

现在再去看下默认的bridge

```
sudo docker network ls
sudo docker network inspect 37c8aaa92fa2
```

![](https://images.notes.xuepincat.com/docker/link/18.png)

也会看到有test2,所以说 test2既连到了默认的docker0上面，也连接到我们自己创建的my-bridge上

另外 inspect my-bridge

![](https://images.notes.xuepincat.com/docker/link/19.png)

很显然，test2和test3这两个容器其实也是可以通的

进入到 test3 中去ping test2 的IP 172.18.0.3

```
sudo docker exec -it test3 /bin/sh
ping 172.18.0.3
```

![](https://images.notes.xuepincat.com/docker/link/21.png)

接着去ping下test2,之前提过如果要ping另一个容器的名称需要通过 `--link` 指定一下到test2上，如果不指定是不能 ping 通的，但是下面的这个是个例外

```
ping test2
```

![](https://images.notes.xuepincat.com/docker/link/22.png)

发现也是可以 ping 通的，这个是因为 test2 和 test3 都连接到了用户自己创建的bridge上面，如果说有两个容器，它连接到用户自己定义的bridge上面，而不是默认的docker0的话，它们两个默认就是已经 link 好了的，也就是说它们两个可以相互通过名字去ping通

现在是在 test3 中，`exit` 退出进入到 test2 中,test2 是既连接docker0的bridge,也连接了用户创建的my-bridge,所以说它有两个地址

```
sudo exec -it test2 /bin/sh
```

![](https://images.notes.xuepincat.com/docker/link/23.png)

现在ping下test3,是可以ping通的

```
ping test3
```

![](https://images.notes.xuepincat.com/docker/link/24.png)

但是现在去 `ping test1` 的时候是不通的

![](https://images.notes.xuepincat.com/docker/link/25.png)

如果现在将test1也连接到my-bridge上，那在test2就可以ping通

先退出 `exit`,将test1 连接到 my-bridge，然后再进入到 test2,去ping下test1

```
exit
sudo docker network connect my-bridge test1
sudo docker exec -it test2 /bin/sh
ping test1
```

![](https://images.notes.xuepincat.com/docker/link/26.png)

![](https://images.notes.xuepincat.com/docker/link/27.png)

通了
+++
title = 'Docker Container'
date = 2019-05-19T17:37:58+08:00
draft = true
categories = [ "Programming" ]
tags = [ "docker" ]
+++

## 什么是Container

* 通过`镜像`创建 (copy)，先有镜像，然后通过镜像创建容器;
* 在 Image Layer 之上建立一个 container layer (可读写),container 是在原先的 image 的基础上增加了一层，叫做 container layer,这一层是可读可写的，而 Image 本身是只读的，container需要运行程序或者安装软件所以需要一个可写的空间;
* 类比面向对象: 类和实例;
* Image 负责 app 的存储和分发，Container 负责运行 app

![](/images/docker/137.jpeg)

## 常用命令

**查看Image**

```shell
docker image list
# 或
docker image ls
```

**查看Container**

```shell
docker container list
# 或
docker container ls
# 可以查看包括已经退出的容器
docker container ls -a
```

**创建Container**

基于Image创建Container。

```shell
docker run <Image Name>
```

图中就可以看到刚才运行的 hello-docker,列表中 COMMAND 一项的值是 `/hello` , 这个是在 Dockerfile 中写的 `CMD ["/hello"]`, 执行的就是 `hello` 的可执行文件，当通过 Image 去创建一个 Container 并运行的时候，它默认会去执行 `CMD` 里面的命令， 因为这个 `CMD` 是一个运行完就结束，不是一个常驻内存的进程，所以容器一旦运行后就退出， `hello` 也就运行结束了

![](/images/docker/143.png)

4. 运行 centos,然后查看运行中的镜像,还是发现什么都没有，使用 `docker container ls -a`

```
docker run centos
docker container ls
docker container ls -a
```

![](/images/docker/144.png)

图中可以看到 centos 对应的 `STATUS` 一栏的值是 `Existed` , `COMMAND` 一栏运行的命令是 `/bin/bash` ，由此可见运行 `/bin/bash` 也不会常驻内存

5. 使用交互式运行

```
docker run -it centos
```

![](/images/docker/145.png)

此时就会进入 centos 的操作系统里面，现在重新打开一个 Terminal ,然后使用命令 `docker container ls` 查看容器

![](/images/docker/146.png)

关于 `-it` 的参数可以使用 `docker run --help` 指令进行查看具体意思

在交互式操作页面有就是进入了 centos ，我们可以进行一些操作，如安装软件， 创建文件等等

6. 可以使用 `exit` 退出，退出之后 Container 将不再运行

![](/images/docker/147.png)

使用 `docker container ls -a` 可以查看刚刚退出运行的容器

![](/images/docker/148.png)

7. 图中可以看到很多退出的容器记录，我们每创建一次容器，就是在 Image 之上多了一层，列表中有多个centos , 这些 centos 的 Container 就是在原先 Cengos Image 基础之上多了一层，使用下面指令将退出的容器删除

```
docker container rm <Container ID>
```

如：

```
docker container rm 01af02e46c67
```

Container ID 其实也不需要写全，也可以写前面的几个字母就行了

如：

```
docker container rm bd
```

![](/images/docker/149.png)


命令 `docker container ls -a` 和 命令 `docker ps -a` 具有相同功能


命令 `dcoker container rm <Container ID>` 和 `docker rm <Container ID>` 具有相同功能

命令 `docker image ls` 和命令 `docker images` 具有相同功能

命令 `docker image rm <Image>` 和命令 `docker rmi <Image>` 具有相同功能

命令 `docker container ls -aq` 和命令 `docker container ls -a | awk {'print$1'}` 效果一样

命令 `docker rm $(docker container ls -aq)` 移除所有退出状态的container

命令 `docker container ls -a1` 和命令 `docker container ls -a | awk {'print$'}` 效果一样，返回第一列的 `Container ID` ， 此时就可以使用命令 docker rm $(docker container ls -aq) 删除所有 Container

命令 `docker container ls -f "status=exited" -q` 列出所有退出的容器

清除命令 `docker rm $(docker container ls -f "status=exited" -q)`

## 进入容器

**进入正在运行中的容器**

```shell
docker exec -it <Container ID> /bin/bash
```

`-it`: 进入交互式界面，并不会执行完程序后就退出

**进入正在运行中的容器，并执行指令**

```shell
docker exec -it {Container ID} {Command}
```

如运行Python、打印IP:
```shell
docker exec -it 662188a370d0 python
docker exec -it 662188a370d0 ip a
```

从容器中退出后容器还会继续运行，可以使用`docker ps`查看容器状态。

## 关闭容器

```shell
docker container stop {Container ID/Name}
或者
docker stop {Container ID/Name}
```

使用 `docker ps -a` 查看所有容器，会发现有很多退出状态的容器，可以使用`docker rm $(docker ps -aq)`清理已经退出状态的容器。

## 容器命名

```shell
docker run -d --name=demo einscat/flask-hello-world
```

## 启动容器

```shell
docker start {Container ID/Name}
```

## 查看容器信息

```shell
docker inspect <Container ID>
```

## 查看日志

```shell
docker logs <Container ID>
```

+++
title = '数据持久化'
date = 2020-07-19T16:01:02+08:00
draft = true
categories = [ "Docker" ]
tags = [ "docker" ]
+++

## Data Volume

有些容器本身会产生一些数据，如果我们不想这些数据随着 Container 的消失而丢失，想保证数据的安全，比如我们使用的数据库，启动一个数据库容器，数据据会有一些数据表等数据，如果把这个 Container 删除了，那么表数据也会随之被删除，这种是不安全的，所以想保留这些数据，此时就会使用到 `Data Volume`

在 Dockerfile 中有个关键字 `volume`,可以通过这个关键字指定该容器中某个目录中产生的数据挂载到 Linux 主机的某个目录中，并且会创建一个叫 `Docker Volume` 的对象

* 创建一个 MySQL 的 Container

```
sudo docker run -d --name mysql1 -e MYSQL_ALLOW_EMPTY_PASSWORD=true mysql
```

```
sudo docker ps
```

```
sudo docker volume ls
```

```
[vagrant@docker-host ~]$ sudo docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                 NAMES
c0b686e7fd40        mysql               "docker-entrypoint.s…"   3 minutes ago       Up 3 minutes        3306/tcp, 33060/tcp   mysql1
[vagrant@docker-host ~]$ sudo docker volume ls
DRIVER              VOLUME NAME
local               6b66c844fb16ad289090c4f54b0bd93034b200159da192f1cb14c07108e223f2
[vagrant@docker-host ~]$
```

![](https://images.notes.xuepincat.com/docker/data-persistence/1.png)

* 查看 volume 具体信息

```
sudo docker volume inspect 6b66c844f...f2
```

![](https://images.notes.xuepincat.com/docker/data-persistence/2.png)

会看到 `volume` 是 `mount` 到本地的 `/var/lib/docker/volumes/6b66c844fb16ad289090c4f54b0bd93034b200159da192f1cb14c07108e223f2/_data` 目录下的

* 创建第二个 MySQL 的 Container

```
sudo docker run -d --name mysql2 -e MYSQL_ALLOW_EMPTY_PASSWORD=true mysql
```

然后再看下 `volume`

```
sudo docker volume ls
```

```
[vagrant@docker-host ~]$ sudo docker run -d --name mysql2 -e MYSQL_ALLOW_EMPTY_PASSWORD=true mysql
0aea00c175b10847c34adab0472e09d3d9073422284610b4a5de8381708a1f77
[vagrant@docker-host ~]$ sudo docker volume ls
DRIVER              VOLUME NAME
local               6b66c844fb16ad289090c4f54b0bd93034b200159da192f1cb14c07108e223f2
local               479711379e0d2007efdcddeac25544be6d835d2629f3f4e9bad74425d4efd2b7
[vagrant@docker-host ~]$
```

![](https://images.notes.xuepincat.com/docker/data-persistence/3.png)

```
sudo docker volume inspect 479711379e0d2007efdcddeac25544be6d835d2629f3f4e9bad74425d4efd2b7
```

![](https://images.notes.xuepincat.com/docker/data-persistence/4.png)

会发现又多了个 volumn

如果将 Container 删掉，对应的 volumn 是不会被删除的

比如将 test1 test2 的 MySQL 容器停掉并关闭

```
sudo docker stop mysql1 mysql2
```

```
sudo docker rm mysql1 mysql2
```

```
sudo docker ps -a
```

但是 volume 还在,这样就解决了数据不会丢失的问题

```
sudo docker volume ls
```

![](https://images.notes.xuepincat.com/docker/data-persistence/5.png)

但是 data volume 的名字并不是特别友好，比如启动两个容器，启动第一个容器的时候产生了一个 volume,但是启动第二个容器的时候还想使用这个 volume, 这个 volume 的名字太长了

其实可以取一个别名

将之前的两个 volume 删除

```
sudo docker volume rm 6b66c844fb16ad289090c4f54b0bd93034b200159da192f1cb14c07108e223f2
```

```
sudo docker volume rm 479711379e0d2007efdcddeac25544be6d835d2629f3f4e9bad74425d4efd2b7
```

* 重新创建 mysql1 的 Container

```
sudo docker run -d -v mysql:/var/lib/mysql --name mysql1 -e MYSQL_ALLOW_EMPTY_PASSWORD=true mysql
```

`-v mysql:/var/lib/mysql`: 取名为 mysql,在本地产生的 data volume (/var/lib/mysql) 映命名成 mysql

![](https://images.notes.xuepincat.com/docker/data-persistence/6.png)

也就是 mysql1 产生的数据都会同步到名叫 mysql 的 volume 中

* 验证

进入 mysql1 

```
sudo docker exec -it mysql1 /bin/bash
```

```
mysql -u root
```

```
show databases;
```

```
create database docker;
```


```
show databases;
```

![](https://images.notes.xuepincat.com/docker/data-persistence/7.png)

然后退出容器，并将 mysql1 停止并删除

```
sudo docker rm -f mysql1
```

`rm -f`: 可以强制删除正在运行中的容器

![](https://images.notes.xuepincat.com/docker/data-persistence/8.png)

但是 volume 还在

```
sudo docker volume ls
```

![](https://images.notes.xuepincat.com/docker/data-persistence/9.png)

下面再去创建一个 MySQL ,并且去使用这个 volume

```
sudo docker run -d -v mysql:/var/lib/mysql --name mysql2 -e MYSQL_ALLOW_EMPTY_PASSWORD=true mysql
```

```
sudo docker exec -it mysql2 /bin/bash
```

```
mysql -u root
```

```
show databases;
```

![](https://images.notes.xuepincat.com/docker/data-persistence/10.png)

会看到 docker 的database还在，也就说明 docker 的 volume 起作用了

## Bind Mouting

Data Volume 可在Dockerfile定义的时候指定持久化的路径,这个路径是在容器中的路径，也就是容器在运行过程中会产生一些文件，这些文件会存在这个路径下面，通过 VOLUME 可以将路径映射到运行这个容器的本地的linux机器的硬盘上

```
VOLUME ["/var/lib/mysql"]
```

```
docker run -v mysql:/var/lib/mysql
```

一种方式是通过 `-v` 的参数将 volume 重新命名一个方便记的名字，比如 mysql

Bind Mouting 和 Data Volume 的区别是 Data Volume 需要在 Dockerfile 中定义需要创建的 volume, 而 Bind Mouting 不需要

只需要在运行容器的时候去指定本地一个目录，和容器中的一个目录一一对应的关系，然后通过这种方式可以做一个同步，也就是本地目录中的文件和运行的容器的目录里的文件是同步的

如果本地的文件做了修改，那么容器中的目录的文件也会做修改，因为它们是同一个文件，同一个目录做了映射

```
docker run -v /home/aaa:/root/aaa
```

* 进入到 labs/docker/nginx 目录

![](https://images.notes.xuepincat.com/docker/data-persistence/11.png)

里面有个 Dockerfile，它通过 nginx 做了一个 base image

并将当前目录的 index.html 文件拷贝到 /usr/share/nginx/html 目录中

这样去启动 Nginx 的 Container 的时候会去本地的 index.html 作为首页

Dockerfile

```
# this same shows how we can extend/change an existing official image from Docker Hub

FROM nginx:latest
# highly recommend you always pin versions for anything beyond dev/learn

WORKDIR /usr/share/nginx/html
# change working directory to root of nginx webhost
# using WORKDIR is prefered to using 'RUN cd /some/path'

COPY index.html index.html

# I don't have to specify EXPOSE or CMD because they're in my FROM
```

* 构建 docker image

```
sudo docker build -t finnley/my-nginx .
```

```
sudo docker image ls
```

![](http://images.notes.xuepincat.com/docker/data-persistence/12.png)

* 创建容器

`-p 80:80`: 将80端口映射到 docker-host 本地80端口

```
sudo docker run -d -p 80:80 --name web finnley/my-nginx
```

```
sudo docker ps
```

![](https://images.notes.xuepincat.com/docker/data-persistence/13.png)

可以看到容器已经运行起来了，并且80已经映射到本地了

* 访问测试

```
curl 127.0.0.1
```

![](https://images.notes.xuepincat.com/docker/data-persistence/14.png)

访问的 index.html 就是在本地的文件，已经将文件拷贝到容器中了，vagrant又把当前80端口映射到本地的8000端口

```
127.0.0.1:8000
```

或者

```
192.168.205.10
```

![](https://images.notes.xuepincat.com/docker/data-persistence/15.png)

![](https://images.notes.xuepincat.com/docker/data-persistence/16.png)

* 将创建的容器停掉并删除

```
sudo docker rm -f web
```

* 重新创建容器，添加 `-v` 参数

将本地的文件映射到容器中，将本地的docker-nginx目录映射到容器中的 `/usr/share/nginx/html` 目录中

```
sudo docker run -d -v $(pwd):/usr/share/nginx/html -p 80:80 --name web finnley/my-nginx
```

* 进入容器

```
sudo docker exec -it web /bin/bash
```

```
ls
```

![](https://images.notes.xuepincat.com/docker/data-persistence/17.png)

* 现在再容器中创建一个 test.txt 的文件后退出

```
touch test.txt
exit
ls
```

![](https://images.notes.xuepincat.com/docker/data-persistence/18.png)

会看到 docker-host 的对应的目录中也多了一个 test.txt 的文件，也就是说当前目录和 /usr/share/nginx/html 目录是同步的

* 修改 test.txt

随便添加些内容，然后保存退出，然后进入到容器中，看下 test.txt

![](https://images.notes.xuepincat.com/docker/data-persistence/19.png)

打开vs code ,进入到 labs/docker-nginx 目录，会看到神奇的一幕，该目录中也多出了一个 test.txt 的文件，并且里面的值也是 abc

![](https://images.notes.xuepincat.com/docker/data-persistence/20.png)

这是因为我机子的 ./labs/ 目录和 vagrant 中的 /home/vagrant/labs 目录是同步的，不管是在哪里改两边都会同步，另外 labs 里面的 docker-nginx 目录又和 container 里面的 /usr/share/nginx/html 目录同步，所以这三个目录是相通的
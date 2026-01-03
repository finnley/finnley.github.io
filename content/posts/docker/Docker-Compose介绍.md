+++
title = 'Docker Compose介绍'
date = 2020-07-22T01:26:28+08:00
draft = true
categories = [ "Docker" ]
tags = [ "docker" ]
+++

## 多容器部署缺点

如果一个 APP 有多个容器组成，去部署的时候会非常繁琐，需要维护多个 Docker Image,也就会遇到下面的一些问题

* 要从 Dockerfile build image 或者 Dockerhub 拉取 image
* 需要创建多个 Container，而且每个 Container 的配置还不一样
* 需要管理这些 Container,比如它们的启动停止删除等
...

所以就想有没有可以批处理的工具，比如可以定义一个文件，在文件中定义 Application 需要的一些 Container，这些定好后使用一条命令就可以完成这些个容器的启动停止删除等操作

## Docker Compose

* Docker Compose 是一个基于 Docker 的命令行工具
* Docker Compose 可以通过一个 `yml` 文件定义多容器的 docker 应用
* 通过一条命令就可以根据 `yml` 文件的定义去创建或管理这些个容器，比如启动，停止，删除等操作

#### docker-compose.yml

`yml` 文件有个默认的名字叫 `docker-compose.yml`，当然也可以自定义，该文件中有三个比较重要的概念 `Services`, `Networks` 和 `Volumes`

#### Services

* 一个 `service` 代表一个 `container`, 这个 `container` 可以从 `dockerhub` 的 `image` 来创建，或者从本地的 `Dockerfile` `build` 出来的 `image` 来创建

* `Service` 的启动类似 `docker run` ,一旦创建了 `service` 就表示通过 `docker run` 创建了一个 `container` , 创建 `container` 的时候可以给其指定 `network` 和 `volume`, 所以可以给 `service` 指定 `network` 和 v`olume` 的引用,也就是可以定义 `network` 和 `volume`,然后在 `sercice` 中引用 `network` 和 `volume`

比如：

```
services:
  db:
    image: postgres:9.4
    volumes:
      - "db-data:/var/lib/postgresql/data"
    networks:
      - back-tier
```

上面定义了一个 `service`, 它的名字叫 `db`, `db` 这个 container 的 image 是通过 dockerhub 拉取下来的， `volumes` 映射了一个 `data-data` 的目录在本地，`db-data` 是通过 `volumes` 来定义的；同理 `networks` 也是一样，定义了一个 `back-tier` 的 networks

上面定义类似下面的语句：

```
docker run -d --network back-tier -v db-data:/var/lib/postgresql/data postgres:9.4
```

再比如：

```
services:
  worker:
    build: ./worker
    links:
      - db
      - redis
    networks:
      - back-tier
```

上面通过 `services` 定义了一个 `container`,这个 `container` 不是从 `dockerhub` 上拉取下来的，而是从本地 `build` 得到的，`./worker` 指定了 `build` `worker` 的 `image` 的 `Dockerfile` 的 `location` 地址；

还定了一个一些 `links`,表示这个 `worker` 容器会和哪几个容器做 `links` ,其实 `links` 没有多大作用，因为都是通过 `docker compose` 创建新的 `docker bridge`,如果所有的 `container` 都是连接在这个 `bridge` 上面的，根本就不需要 `links`,只有连接默认的 `docker bridge` 的时候才会使用 `links` 去做 DNS 的翻译

#### Volumes

```
services:
  db:
    image: postgres:9.4
    volumes:
      - "db-data:/var/lib/postgresql/data"
    networks:
      - back-tier

volumes:
  db-data
```

仍然使用上面的例子，使用到了 `db-data` 的 `volume`,另外会在和 `services` 的同一级定义一个 `volumes`, `volumes` 的下级定义 `db-data`

类似如下命令：

```
docker volume create db-data
```

#### Networks

```
services:
  worker:
    build: ./worker
    links:
      - db
      - redis
    networks:
      - back-tier

networks:
  front-tier:
    driver: bridge
  back-tier:
    driver: bridge
```

同理 `networks` 也是一样，比如有个 `networks` 叫做 `back-tier`,这个 `back-tier` 就是在与 `services` 同级的 `networks` 里面定义的，`back-tier` 的 `driver` 是 `bridge`,也就是会通过 `docker networks` 去创建一个 `driver` 是 `bridge` 并且名字是 `back-tier` 的 `network`

类似下面语句：

```
docker network create -d bridge back-tier
```
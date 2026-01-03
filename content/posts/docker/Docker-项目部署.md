+++
title = '项目部署'
date = 2020-07-20T21:51:11+08:00
draft = true
categories = [ "Docker" ]
tags = [ "docker" ]
+++

## Bind Mount 实时更新

通过Dockerfile将一个flask的application构建成docker image

Dockerfile

```
FROM python:2.7
LABEL maintainer="Finnley<mingmin.yuen@gmail.com>"

COPY . /skeleton
WORKDIR /skeleton
RUN pip install -r requirements.txt
EXPOSE 5000
ENTRYPOINT ["scripts/dev.sh"]
```

`COPY . /skeleton`: 将当前目录的源代码拷贝到容器的根目录 `skeleton`
`WORKDIR /skeleton`: 切换目录
`RUN pip install -r requirements.txt` 安装依赖
`EXPOSE 5000`: 对外暴露5000端口
`ENTRYPOINT ["scripts/dev.sh"]`: 运行脚本文件

这个环境是本机，已经安装好 Docker

![](https://images.notes.xuepincat.com/docker/cases/14.png)

* 进入源码目录,构建 image

```
docker build -t finnley/flask-skeleton .
```

```
docker image ls
```

![](https://images.notes.xuepincat.com/docker/cases/15.png)

* 通过 image 创建 Container

```
docker run -d -p 8000:5000 --name flask-skeleton finnley/flask-skeleton
```

```
docker ps 
```

![](https://images.notes.xuepincat.com/docker/cases/16.png)

* 测试

在浏览器中输入 `127.0.0.1:8000`,回车

![](https://images.notes.xuepincat.com/docker/cases/17.png)

比如现在开发者想对程序做些修改，修改完后想立刻看下效果，一种方法是重新build这个项目的 image,然后重新运行，这种方法比较不方便

另外一种方式通过 bind mount 可以实时同步源文件，就可以实时看到修改效果

* 删除之前运行中的容器

```
docker rm -f flask-skeleton
```

* 重新创建 Container,并将目录映射到容器中

```
docker run -d -p 8000:5000 -v $(pwd):/skeleton --name flask-skeleton finnley/flask-skeleton
```

`$(pwd)`: 表示当前目录
`/skeleton`: 容器中的目录

```
docker ps
```

![](https://images.notes.xuepincat.com/docker/cases/18.png)

* 打开源码

将原来的 `Welcome` 修改为 `Welcome to Docker World`,刷新一下页面就看到页面内容改变了

![](https://images.notes.xuepincat.com/docker/cases/19.png)

## 部署 wordpress

* 拉取 wordpress 到本地

```
docker pull wordpress
```

![](https://images.notes.xuepincat.com/docker/cases/20.png)

* 拉取 mysql5.7 到本地

```
docker pull mysql:5.7.31
```

![](https://images.notes.xuepincat.com/docker/cases/21.png)

* 创建 MySQL 容器

```
docker run -d --name mysql -v mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=wordpress mysql:5.7.31
```

![](https://images.notes.xuepincat.com/docker/cases/22.png)

* 创建 wordpress 容器

```
docker run -d -e WORDPRESS_DB_HOST=mysql:3306 --link mysql -p 8080:80 wordpress
```

`mysql:3306`: 之前启动的 Container 名称是 mysql 的容器
`--link mysql`: link 到 mysql
`-p 8080:80`: 将 wordpress 容器里面的80端口映射到本地8080

![](https://images.notes.xuepincat.com/docker/cases/23.png)

* 测试

打开浏览器，输入 `127.0.0.1:8080` ，就可以看到安装向导页面了

![](https://images.notes.xuepincat.com/docker/cases/24.png)

设置信息后就看到下面页面了

![](https://images.notes.xuepincat.com/docker/cases/25.png)

到这里 docker 搭建的 wordpress 就部署完成了

## 部署投票系统

1. 项目介绍

![](https://images.notes.xuepincat.com/docker/cases/architecture.png)

部署的 Application 分为5个模块，有两个模块是 web application,需要暴露给外部访问的web应用，一个是 Voting App,用于用户投票使用，一个是 Result APP,用于投票结果显示。

Voting App 连接的是 Redis 缓存，因为 Voting App 访问量很大，对于访问量很大的投票需要先将投票结果写入到缓存中，后面有个 Java Worker 去取结果再将结果写到数据库中。

Result App 直接去数据库中获取最新的投票结果。


Result App  是一个 JavaScript 项目，Voting App 是一个 Python Flask 项目，Java Worker 是一个 Java 项目

2. docker-compose.yml

```
version: "3"

services:
  voting-app:
    build: ./voting-app/.
    volumes:
     - ./voting-app:/app
    ports:
      - "5000:80"
    links:
      - redis
    networks:
      - front-tier
      - back-tier

  result-app:
    build: ./result-app/.
    volumes:
      - ./result-app:/app
    ports:
      - "5001:80"
    links:
      - db
    networks:
      - front-tier
      - back-tier

  worker:
    build: ./worker
    links:
      - db
      - redis
    networks:
      - back-tier

  redis:
    image: redis
    ports: ["6379"]
    networks:
      - back-tier

  db:
    image: postgres:9.4
    volumes:
      - "db-data:/var/lib/postgresql/data"
    networks:
      - back-tier

volumes:
  db-data:

networks:
  front-tier:
  back-tier:
```

3. 启动

```
docker-compose up
```

4. 访问测试

投票

`127.0.0.1:5000`

结果

`127.0.0.1:5001`

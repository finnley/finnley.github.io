+++
title = 'docker-compose安装与使用'
date = 2020-07-23T05:26:28+08:00
draft = true
categories = [ "Docker" ]
tags = [ "docker" ]
+++

## 安装

Mac 和 Window 系统在安装 Docker 的时候 Compose 就已经一起安装了，只有 Linux 系统需要另外安装

[Install Docker Compose](https://docs.docker.com/compose/install/)

* Run this command to download the current stable release of Docker Compose:

```
sudo curl -L "https://github.com/docker/compose/releases/download/1.26.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

* Apply executable permissions to the binary:

```
sudo chmod +x /usr/local/bin/docker-compose
```

* You can also create a symbolic link to /usr/bin or any other directory in your path. (Optional)

```
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

* Test the installation.

```
docker-compose --version
```

![](https://images.notes.xuepincat.com/docker/compose/1.png)

## 案例 - wordpress

* docker-compose.yml

```
version: '3'

services:

  wordpress:
    image: wordpress
    ports:
      - 8080:80
    depends_on:
      - mysql
    environment:
      WORDPRESS_DB_HOST: mysql
      WORDPRESS_DB_PASSWORD: root
    networks:
      - my-bridge

  mysql:
    image: mysql:5.7
    ports:
      - 33060:3306
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: wordpress
    volumes:
      - mysql-data:/var/lib/mysql
    networks:
      - my-bridge

volumes:
  mysql-data:

networks:
  my-bridge:
    driver: bridge
```

* 第一行 `version: '3'` 表示使用的 `Docker compose version 3`;
* 下面有三大部分 `services`, `volumes`, `networks`;
* `servies` 中定义了两个 `sevice`, `wordpress` 和 `mysql`；
* `wordpress` 使用的 `image` 是 `wordpress`,然后做了端口映射，将 `wordpress` 的 80 端口映射到本地 `8080`;
* 然后设置 `environment` ，相当于通过 `-E` 参数来传递环境变量，一个是 mysql 的 host ,一个是 mysql 的密码;
* `networks` 指定了容器的连接的网络是 `my-bridge`,也就是最下面的 `bridge network`;
* `mysql` 同样定义了 `image`, `environment`,并且将 `mysql` 的 `volume` 映射成 `mysql-data`,这个 `volumes` 也就是在下面 `volumes` 创建的，同理还有 `networks`

这里是在本地环境使用 wordpress 和 mysql 搭建项目，当前目录是在 `wordpress` 目录下

1. 启动

启动 `yml` 文件中定义的 `service` 的 `container` 

- 如果不输入任何参数，默认的启动文件就是 `docker-compose.yml`

```
docker-compose up
```

如果使用 `docker-compose up` 启动的时候界面会一直停留，需要使用 `CTRL + C` 取消，这种方式一般作为 DEBUG 执行

其实可以使用 `-d` 参数在后台执行，但是启动的container log 不会输出出来

```
docker-compose up -d
```

- 指定参数,指定启动哪个 docker compose up,因为默认的启动文件就是 `docker-compose.yml`，所以可以直接使用 `docker-compose up`

```
docker-compose -f docker-compose.yml up
```

2. 常用命令

* 关闭

```
docker-compose stop
```

* 停止并删除，但是并不会删除本地的 docker image

```
docker-compose down
```

* 启动

```
docker-compose start
```

* 查看

```
docker-compose ps
```

* 查看 docker-compose 定义的 container 和所使用的 image

```
docker-compose images
```

* docker-compsose exec [ServiceName]

比如

```
docker-compose exec mysql bash
```

* 查看网络

```
docker network ls
```

## 案例 - 使用 Dockerfile 构建镜像

* Dockerfile

```
FROM python:2.7
LABEL maintaner="mingmin.yuen@gmail.com"
COPY . /app
WORKDIR /app
RUN pip install flask redis
EXPOSE 5000
CMD [ "python", "app.py" ]
```

有一个 `python` 的应用，使用 `Dockerfile` 去 `build` flask 成为 `docker image`，接着定义一个 `docker-compose.yml`，内容如下：

```
version: "3"

services:

  redis:
    image: redis

  web:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - 8080:5000
    environment:
      REDIS_HOST: redis
```

可以看到有两个 `services`, 一个是 `redis`, `redis` 从 `dockerhub` 上拉取就可以创建一个 `redis` 的容器；`web` 的 `servies` 不是从 `dockerhub` 上拉取的，而是去构建的，也就是传入的是 `Dockerfile` 的位置和名字，通过 `Dockerfile` build image 出来，然后通过 `build` 的 `image` 去创建 `Container`

进入到 `flask-redis` 目录

1. 启动

```
docker-compose up
```

2. 测试

在浏览器输入 `http://127.0.0.1:8080/`

![](https://images.notes.xuepincat.com/docker/compose/2.png)

## 案例 - scale

1. 后台启动运行

```
docker-compose up -d
```

2. 查看运行中的服务

```
docker-compose ps
```

![](https://images.notes.xuepincat.com/docker/compose/3.png)

3. scale

从上图中可以看到每个 `redis` 和 `web` 的容器只有一个，通过 `scale` 可以进行扩展，比如现在 `web` 只有一个，可以通过 `scale` 将 `web` 扩展成3个

```
docker-compose up --scale web=3 -d
```

但是执行的时候会报错

```
ERROR: for flask-redis_web_3  Cannot start service web: driver failed programming external connectivity on endpoint flask-redis_web_3 (ecfe02038d65f29c787608e7abfc88aa584887abdf171d39d4704692709b11bb): Bind for 0.0.0.0:8080 failed: port is already allocated

ERROR: for flask-redis_web_2  Cannot start service web: driver failed programming external connectivity on endpoint flask-redis_web_2 (344a1e69786476b692b60df0660d90fbdf66c4eb8270bd100b7746eb2f9edfb4): Bind for 0.0.0.0:8080 failed: port is already allocated

ERROR: for web  Cannot start service web: driver failed programming external connectivity on endpoint flask-redis_web_3 (ecfe02038d65f29c787608e7abfc88aa584887abdf171d39d4704692709b11bb): Bind for 0.0.0.0:8080 failed: port is already allocated
```

![](http://images.notes.xuepincat.com/docker/compose/4.png)

这是因为 `8080` 的端口已经被分配了

现将之前启动的停止并删除

```
docker-compose down
```

上面的 Dockerfile 中web容器的5000端口绑定了本地的8080,如果再去 scale web 容器的时候启动三个web,每个 web 的端口都绑定到8080是不可能，因为本地只有一个8080,所以是这一块导致失败的原因

可以将 ports 删除

修改后的 docker-compose.yml

```
version: "3"

services:

  redis:
    image: redis

  web:
    build:
      context: .
      dockerfile: Dockerfile
    environment:
      REDIS_HOST: redis
```

然后重新执行上面的步骤

```
docker-compose up -d
```

```
docker-compose ps
```

```
docker-compose up --scale web=3 -d
```

![](https://images.notes.xuepincat.com/docker/compose/5.png)

图中可以看到有三个 `web` 的 `container`,它们三个都是监听了 `5000` 端口，这里的 `5000` 都是容器的 `namespace` 的 `5000` 端口，并没有映射到主机的 `5000`，现在就有了三个web服务，并且都去访问redis

![](https://images.notes.xuepincat.com/docker/compose/6.png)

比如现在有n个web服务，它们都监听本地8000, 8001, ...等一些端口，并且它们同时去访问一个redis,如果前面再加个 `HAProxy` 负载均衡器，能够将外界进来的访问量平均的负载到各个 `web` 上，这样就可以减少单个服务器的访问压力，这样就可以实现一种好的部署访问模式，并且使用这种方式去部署访问的话，未来的扩展性是非常强的,比如现在有三台负载均衡器去做负载均衡，如果有一天发现冲突了，比如访问量太大，现在打算添加机器，如果使用 `docker-compose` 可以很方便的进行 `scale`,就可以很快将 `web` 服务扩展到10个或者其他数量

4. 再扩展

比如现在的web是3个，我想再扩展到10个

```
docker-compose up --scale web=10 -d
```

![](https://images.notes.xuepincat.com/docker/compose/7.png)

#### 案例 - HAProxy

现在有个 `lb-scale` 项目

* app.py

```
from flask import Flask
from redis import Redis
import os
import socket

app = Flask(__name__)
redis = Redis(host=os.environ.get('REDIS_HOST', '127.0.0.1'), port=6379)


@app.route('/')
def hello():
    redis.incr('hits')
    return 'Hello Container World! I have been seen %s times and my hostname is %s.\n' % (redis.get('hits'),socket.gethostname())


if __name__ == "__main__":
    app.run(host="0.0.0.0", port=80, debug=True)
```

* Dockerfile

```
FROM python:2.7
LABEL maintaner="mingmin.yuen@gmail.com"
COPY . /app
WORKDIR /app
RUN pip install flask redis
EXPOSE 80
CMD [ "python", "app.py" ]
```

端口是80

* docker-compose.yml

```
version: "3"

services:

  redis:
    image: redis

  web:
    build:
      context: .
      dockerfile: Dockerfile
    environment:
      REDIS_HOST: redis

  lb:
    image: dockercloud/haproxy
    links:
      - web
    ports:
      - 8080:80
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock 
```

1. 将之前启动的关系并停止

```
docker-compose down
```

2. 进入 `lb-scale` 目录， 启动

```
docker-compose up -d
```

```
docker-compose ps
```

![](https://images.notes.xuepincat.com/docker/compose/8.png)

现在启动了一个 `redis` 服务，一个 `web` 服务，还有一个 `haproxy`

3. 测试访问

```
curl 127.0.0.1:8080
```

![](https://images.notes.xuepincat.com/docker/compose/9.png)

hostname 就是 `web_1` 容器的 hostname

4. scale 扩展

```
docker-compose up --scale web=3 -d
```

```
docker-compose ps
```

![](https://images.notes.xuepincat.com/docker/compose/10.png)

5. 多次访问测试,

```
curl 127.0.0.1:8080
```

![](http://images.notes.xuepincat.com/docker/compose/11.png)

会将请求进行轮询，将请求转发到 `web_1` , `web_2` , `web_3`

6. 扩展到5个 web 服务

```
docker-compose up --scale web=5 -d
```

```
docker-compose ps
```

7. 循环请求

```
for i in `seq 10`; do curl 127.0.0.1:8080; done
```

![](https://images.notes.xuepincat.com/docker/compose/12.png)

## docker-compose build

如果 `docker-compose` 里面定义的 `service` 有些 `image` 需要通过 `Dockerfile` 构建的，就可以先通过 `docekr-compose build` 把需要的 `image` 先构建下，然后再去启动 (up) 就会变的快一点，如果直接去up,它就先去 `build` 在 `up`,一步做了两步的事情；

另外如果 `services` 的 `Dockerfile` 发生了变化，就需要通过 `docker-compose build` 重新构建 `image`,然后再通过 `docker-compose up` 启动最新的 `service`;

`docekr-compose` 通常用于本地开发的工具，通常不是用于部署生产环境的工具，生产环境通常使用 `k8s`
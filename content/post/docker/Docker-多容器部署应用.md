+++
title = '多容器部署应用'
date = 2020-05-06T23:45:55+08:00
draft = true
categories = [ "Docker" ]
tags = [ "docker" ]
+++

* 背景

有个项目叫 flask-redis ,里面有个 hello 函数，该作用是当访问该 url 时，redis中会有个key,这个key每次都是会增加的，比如第一次访问的时候值就是1，并且会返回一句话，第二次访问的时候key的值会自动加1

app.py 

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
    app.run(host="0.0.0.0", port=5000, debug=True)

```

Dockerfile

```
FROM python:2.7
LABEL maintaner="finnley@gmail.com"
COPY . /app
WORKDIR /app
RUN pip install flask redis
EXPOSE 5000
CMD [ "python", "app.py" ]
```

使用 python2.7 的 base image；
然后将源码拷贝到根目录 app 下面；
使用 WORKDIR 改变工作目录；
安装 flask 和 redis；
暴露 5000 端口，因为 app.py 中绑定了5000端口，后面需要通过 `-p` 参数将 5000 端口map出来；
启动 python

现在需要启动一个 redis 的 server,但是在 Dockerfile 里面并没有安装 redis 和 启动 redis,而是安装了一个redis的 python 库,然后使用这个库去连接 redis

所以这里将 flask 本身作为一个 app container 进行部署,redis 也单独作为一个 Container,它们之间有个互相访问的关系，就是 flask 这个 Container 需要去访问 redis 这个 Container

* 创建一个正在后台执行的 redis Container

```
sudo docker run -d --name redis redis
```

![](https://images.notes.xuepincat.com/docker/cases/1.png)

这边创建 redis 的时候并没有提供 `-p` 参数，因为这个redis并不是供外部访问的，而是供app内部访问的，这个只是供 flask 访问，所以没有必要将 6389 端口暴露到外面

flask 访问 redis 的时候并不知道 redis 的IP地址，app.py 中连接的 redis 地址是通过环境变量 (REDIS_HOST) 的方式获取到的

但是现在 redis 是一个容器，并不知道这个redis的IP地址，虽然可以查看到redis的IP地址，另外可以通过LINK的方式在启动第二个容器的时候直接通过redis的名字就可以访问到container

* 构建 flask-redis 的 image

```
sudo docker build -t finnley/flask-redis .
```

* 查看

```
sudo docker image ls
```

![](https://images.notes.xuepincat.com/docker/cases/2.png)

* 创建 Container

```
sudo docker run -d --link redis --name flask-redis -e REDIS_HOST=redis finnley/flask-redis
```

![](https://images.notes.xuepincat.com/docker/cases/3.png)

--link redis: 在该容器中可以直接通过 redis 这个名字访问 redis Container
--name flask-redis: 容器名字为 flask-redis
-e REDIS_HOST=redis finnley/flask-redis: 在要启动的容器里面设置了一个环境变量叫 REDIS_HOST ，该环境变量的值叫 redis,然后最后是镜像 Image 的名称

* 进入 Container

```
sudo docker exec -it flask-redis /bin/bash
```

![](https://images.notes.xuepincat.com/docker/cases/4.png)

输入 `env`,会看到里面有个 `REDIS_HOST=redis`

![](https://images.notes.xuepincat.com/docker/cases/5.png)

并且这个容器是可以直接去 `ping redis`

![](https://images.notes.xuepincat.com/docker/cases/6.png)

源码中是直接获取 REDIS_HOST 的环境变量的，这个环境变量就是 redis,也就是 `redis:6379`

* 测试 

```
curl 127.0.0.1:5000
```

![](https://images.notes.xuepincat.com/docker/cases/7.png)

再多请求几次

![](https://images.notes.xuepincat.com/docker/cases/8.png)

现在是在 Container 里面，如果到 Container 外面呢？

```
exit
sudo docker ps
```

![](https://images.notes.xuepincat.com/docker/cases/9.png)

会看到 5000 端口并没有暴露出来，所以访问的时候访问不了

```
curl 127.0.0.1:5000
```

![](https://images.notes.xuepincat.com/docker/cases/10.png)

先将 flask-redis 停掉,并删掉

```
sudo docker stop flask-redis
```

```
sudo docker rm flask-redis
```

* 重新启动 flask-redis,并添加 `-p` 参数

```
sudo docker run -d -p 5000:5000 --link redis --name flask-redis -e REDIS_HOST=redis finnley/flask-redis
```

![](https://images.notes.xuepincat.com/docker/cases/11.png)

* 然后再去访问

```
curl 127.0.0.1:5000
```

![](https://images.notes.xuepincat.com/docker/cases/12.png)

这样就实现了两个 Container 相互访问的关系

* 环境变量

创建了一个test1的container，并且设置了一个环境变量 `MY_NAME`,值为 `finnley`

```
sudo docker run -d --name test1 -e MY_NAME=finnley busybox /bin/sh -c "while true; do sleep 3600; done"
```

进入到 test1 容器

```
sudo docker exec -it test1 /bin/sh
```

查看环境变量

```
env
```

![](https://images.notes.xuepincat.com/docker/cases/13.png)

会看到环境变量中有个 `MY_NAME=finnley`

可以通过这种方式提前给要创建的容器设置环境变量
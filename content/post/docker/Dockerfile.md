+++
title = 'Dockerfile'
date = 2019-05-26T13:29:37+08:00
draft = true
categories = [ "Docker" ]
tags = [ "docker" ]
+++

# 语法

## FROM

尽量使用官方的 image 作为base image !

* FROM scratch # 制作base image
* FROM centos # 使用base image
* FROM ubuntu:16.04

## LABEL

Metadata 不可少，有点像代码中的注释

* LABEL maintainer="email@example.com"
* LABEL version="1.0"
* LABEL description="This is description"

## RUN

每运行一次 run ,对于 image 都会生成新的一层，所以对于 run ,为了美观,复杂的 RUN 请用反斜线换行！避免无用分层，合并多条命令成一行！

```
RUN yum update && yum install -y vim \
python-dev #反斜线表示换行
```

```
RUN apt-get update && apt-get install -y perl \
pwgen --no-install-recommends && rm -rf \
/var/lib/apt/lists/* #注意清理cache
```

```
RUN /bin/bash -c 'source $HOME/.bashrc;echo $HOME'
```

## WORKDIR

* 设定当前工作目录，像 Linux 中的 `cd` 命令改变目录，然后在当前目录中做一些事情，如运行一些程序，或者创建一些文件或者目录
* 使用 `WORKDIR`，不要用 `RUN cd` ! 
* 尽量使用绝对目录，

```
WORKDIR /root
```

```
WORKDIR /test # 如果没有会自动创建test目录
WORKDIR demo
RUN pwd # 输出结果应该是 /test/demo
```

## ADD and COPY

* 大部分情况下，`COPY` 优于 `ADD` ！
* `ADD` 除了 `COPY` 还有额外功能 （解压）！
* 添加远程文件或者目录使用 `curl` 或者 `wget` !
* `ADD` 和 `COPY` 非常像，都是将本地文件添加到image里面

```
ADD hello / # 将当前目录中的 hello 可执行文件 ADD 到 image 根目录中
```

```
ADD test.tar.gz / # 添加到根目录并解压 
```

```
WORKDIR /root
ADD hello test/ # /root/test/hello
```

```
WORKDIR /root
COPY hello test/
```

## ENV

尽量使用 `ENV` 增加 Dockerfile 可维护性！

```
ENV MYSQL_VERSION 5.6 # 设置常量
RUN apt-get install -y mysql-server="${MYSQL_VERSION}" \
&& rm -rf /var/lib/apt/lists/* # 引用常量
```

## RUN

执行命令并创建新的 `Image Layer`

## CMD

设置容器启动后默认执行的命令和参数

* 容器启动时默认执行命令
* 如果 `docker run` 指定了其他命令，CMD命令被忽略
* 如果定义了多个 CMD，只有最后一个会执行

如：

```
FROM centos
ENV name Docker
CMD echo "hello $name"
```

当执行 `docker run [image]` 的时候会输出 `hello Docker`,满足第一个条件;
当执行 `docker run -it [image] /bin/bash` 的时候，满足第二个条件，此时 CMD 命令被忽略，将不会输出 `hello Docker`;
当定义了很多个 CMD，只有最后一个会执行；

## ENTRYPOINT 

设置容器启动时运行的命令

* 让容器以应用程序或者服务的形式运行,一般作为后台的进程，如启动数据库的服务
* 不会被忽略，一定会执行
* 最佳实践：写一个 shell 脚本作为entrypoint

![](https://images.notes.xuepincat.com/docker/164.png)

如：

```
COPY docker-entrypoint.sh /usr/local/bin
ENTRYPOINT ["docker-entrypoint.sh"]

EXPOSE 27017
CMD ["mongod"]
```

## Shell 和 Exec 格式

1. Shell 格式

把要运行的命令当做 shell 命令去执行

```
RUN apt-get install -y vim
CMD echo "hello docker"
ENTRYPOINT echo "hello docker"
```

2. Exec 格式

```
RUN ["apt-get", "install", "-y", "vim"]
CMD ["/bin/echo", "hello docker"]
ENTRYPOINT ["/bin/echo", "hello docker"]
```

#### Dockerfile1

```
FROM centos
ENV name Docker
ENTRYPOINT echo "hello $name"
```

结果：

![](https://images.notes.xuepincat.com/docker/161.png)

#### Dockerfile2

```
FROM centos
ENV name Docker
ENTRYPOINT ["/bin/echo", "hello $name"]
```

![](https://images.notes.xuepincat.com/docker/162.png)

使用 `exec` 格式并没有将 `$name` 替换成 `ENV` 中的定义的常量，这是为什么呢？

通过 `shell` 格式运行命令的时候，默认会通过shell,如所在的Linux,在bash里面，通过shell去执行命令，所以它会识别 `$name` 是一个变量，把变量替换成对应的值，所以就会打印 `hello Docker`;

通过 `exec` 格式 `echo` 打印字符串的时候只是通过 `echo` 执行后面的命令，并不是 `shell` ,也就是并不是在 `shell` 里面执行 `echo` ,而是单纯的执行 `echo` ,并没有将 `$name` 替换掉

修改 `Dockerfile` ,让 `exec` 格式的命令被 `shell` 执行,需要在Dockerfile里面指明要运行的命令是通过shell去运行的

```
FROM centos
ENV name Docker
ENTRYPOINT ["/bin/bash", "-c", "echo hello $name"]
```

![](https://images.notes.xuepincat.com/docker/163.png)

# 练习

## Dockerfile打包Python程序

1. 创建 `flask-helo-docker` 的目录

```
mkdir flask-hello-docker
```

2. 在目录下创建 `app.py` 文件, 并编辑

```
from flask import Flask
app = Flask(__name__)
@app.route('/')
def hello():
    return "hello docker"
if __name__ == '__main__':
    app.run()
```

`:wq` 保存退出

3. 创建 `Dockerfile`

```
cd flask-hello-docker
vim Dockerfile
```

4.编辑 `Dockerfile`, 将 `app.py` 添加到 Image 中， 继续编辑 Dockerfile

```
FROM python:2.7
LABEL maintainer="Finnley<mingmin.yuen@gmail.com>"
RUN pip install flask
COPY app.py /app/
WORKDIR /app
EXPOSE 5000
CMD ["python", "app.py"]
```

`：wq` 保存退出

6. 构建 Image

```
docker build -t finnley/flask-hello-docker .
```

7. 此时已经有了新的 Image ， 可以通过下面命令进行查看

```
docker image ls
```

![](/images/docker/178.png)

8. 通过 Image 创建 Container

```
docker run finnley/flask-hello-docker 
```

![](/images/docker/179.png)

但是此时是占用屏幕了，如果使用 `CTRL + C` 就会退出， 可以使用下面方法在后台运行

```
docker run -d finnley/flask-hello-docker
```

![](/images/docker/180.png)

使用下面命令查看 Container 状态

```
docker ps
```

![](/images/docker/181.png)

## 打包 Stress 为 Image

1. 创建一个 Ubuntu 的容器

```
cd ~
mkdir ubuntu-stress
cd ubuntu-stress
docker run -it ubuntu
```

![](/images/docker/188.png)

2. 在容器中安装 `stress`

```
apt update && apt install -y stress
```

3. 查看 `stress` 所在位置

```
which stress
```

![](/images/docker/189.png)

可以通过 `stress --help` 查看帮助信息

```
stress --help
```

![](/images/docker/190.png)

4. `stress` 简单使用

```
stress --vm 1
```

启动一个 work (进程), 默认分类 256M 内存

![](/images/docker/191.png)

此时现在界面中没有任何输出，我们可以添加 `--verbose` 参数在屏幕中展示 debug 输出

```
stress --vm 1 --verbose
```

![](/images/docker/192.png)

从 debug 中可以看到创建了一个进程，并分配一些内存，然后再进行释放，如此循环往复

`--vm-bytes` 参数默认分类 256M 内存，现在分配 50000M 内存

```
stress -vm 1 --vm-bytes 50000M --verbose
```

![](/images/docker/193.png)

此时分配的 50000M 内存已经超出了限制

暂时退出，查看 docker host 内存信息

```
exit
top
```

![](/images/docker/194.png)

图中可以看到 `total` 总内存有 498888KiB , 所以在 docker host 上创建的容器内存最大不会超过宿主机内存

5. 在当前目录中创建 Dockerfile ,并进行编辑

```
FROM ubuntu
RUN apt-get update && apt-get install -y stress
ENTRYPOINT ["/usr/bin/stress"]
CMD []
```

6. `:wq` 暂时保存退出，先进行构建

```
docker build -t finnley/ubuntu-stress .
docker image ls
```

![](/images/docker/195.png)

![](/images/docker/196.png)

7. 运行 

```
docker run -it finnley/ubuntu-stress
```

![](/images/docker/197.png)

此时结果和就在 ubuntu 里面运行 `stress` 命令一样

现在如果想要运行，并且可以执行参数，怎么做呢？ 比如下面

```
docker run -it finnley/ubuntu-stress --vm 1
```

![](/images/docker/198.png)

同理

```
docker run -it finnley/ubuntu-stress --vm 1 --verbose
```

后面跟的参数是在放在 Dockerfile 中 `CMD []` 里面的， 我们通过 `ENTRYPOINT` 运行 `stress` , 后面的参数也就是传进来的如 `--vm 1` 就是通过 `CMD []` 接收的， 也可以将 `CMD []` 里面添加一个默认的参数， 如 `CMD ["--verbose"]`

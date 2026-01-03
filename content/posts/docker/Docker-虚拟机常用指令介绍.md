+++
title = 'Docker虚拟机常用指令介绍'
date = 2018-09-23T00:19:16+08:00
draft = true
categories = [ "Docker" ]
tags = [ "docker" ]
+++

## CentOS安装Docker

```
yum update -y
yum install -y docker
```

## 常用管理命令

启动、关闭、重启

```
service docker start
service docker stop
service docker restart
```

![](https://images.notes.xuepincat.com/docker/2.png)

左上角的 `DockerFile` 文件定义了镜像要安装程序和配置的环境，通过 `build` 指令可以创建出我们想要的镜像；

一旦创建出镜像，如果想要将镜像分发给其他主机的Docker虚拟机，一种方法是借助Docker仓库来实现，我们可以通过 `push` 指令把本地镜像上传到仓库中，其他主机就可以通过 `search` 指令到仓库中去查找上传的镜像，找到上传的镜像之后可以通过 `pull` 指令将镜像下载到本地，这样别的主机的Docker虚拟机就可以拥有这个镜像了；另一种方式是通过文件的方式，将镜像导出为压缩文件，别的主机再用压缩文件导入为镜像就可以了，导出指令是 `save` 或 `export`，导入的指令是 `load` 或 `import`;

镜像一旦创建出来也是可以删除的，通过 `rmi` 指令可以将镜像删除；

如果想要查看镜像的详细信息，可以使用 `inspect` 指令；

如果想要查看Docker虚拟机内的所有镜像，可以使用 `images` 指令；

镜像是用来创建容器的，从镜像创建出容器的指令是 `run`；

创建出容器之后，这个容器就直接运行了，如果想要停止容器运行或者删除容器，可以使用指令 `pause` 指令暂停，如果恢复运行可以使用 `unpause` 指令；如果想要彻底停止容器的指令是 `stop` ,恢复运行指令为 `start`;

查看容器详细信息可以使用指令 `inspect`;

如果想要查看Docker虚拟机内的所有所有容器可以使用 `ps` 指令，如果删除容器可以使用 `rm` 指令；

容器可以保存为镜像，在容器里面安装程序，配置环境，然后保存为镜像，可以使用 `commit` 指令。

### 安装Java镜像

```
docker search java # 搜索与java相关的的镜像
docker pull java # 下载指定的镜像
```

国外镜像仓库下载速度较慢，建议使用国内镜像仓库，如 `DaoCloud`， [<span style="color:#FF1493;">DaoCloud镜像配置</span>](https://www.daocloud.io/mirror) ， 找到Linux的配置，将其复制粘贴到终端

* Linux下配置

```
curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://f1361db2.m.daocloud.io
```

配置完后还需要修改配置文件

```
vim /etc/docker/daemon.json
```

修改前：

```
{"registry-mirrors": ["http://f1361db2.m.daocloud.io"],}
```

将最后的 `,` 逗号去掉就行了

修改后：

```
{"registry-mirrors": ["http://f1361db2.m.daocloud.io"]}
```

接下来搜索与java有关的镜像

```
docker search java
```

![](https://images.notes.xuepincat.com/docker/3.png)

这里使用的是 `docker.io/java` ，将镜像的名称复制粘贴到下面代码中

![](https://images.notes.xuepincat.com/docker/4.png)

```
docker pull docker.io/java
```

![](https://images.notes.xuepincat.com/docker/5.png)

然后使用下面命令显示docker里面安装的镜像是什么

```
docker images
```

![](https://images.notes.xuepincat.com/docker/6.png)

### 导出导入镜像

安装了Docker镜像，如果想备份镜像把镜像导出，保存为压缩文件，用到的指令是 `save`。如果要从压缩文件导入镜像，使用的指令是 `load`。

#### 语法

```
docker save java > /home/java.tar.gz #导出镜像 
docker load < /home/java.tar.gz #导入镜像
docker images #查看docker虚拟机里面导入导出的镜像有哪些
docker rmi java #删除镜像
```

#### 实操

* 导出刚才安装的java镜像 

![](https://images.notes.xuepincat.com/docker/7.png)

* 查看一下是否导出成功

![](https://images.notes.xuepincat.com/docker/8.png)

* 现在将docker虚拟机里面的镜像删掉

![](https://images.notes.xuepincat.com/docker/9.png)

* 查看镜像

![](https://images.notes.xuepincat.com/docker/10.png)

* 导入镜像

![](https://images.notes.xuepincat.com/docker/11.png)

* 查看镜像

![](https://images.notes.xuepincat.com/docker/12.png)

### 启动容器

#### 语法

```
docker run ...
```

#### 示例

```
docker run -it --name myjava java bash
```

* -it: 启动容器之后开启一个交互的界面
* --name: 给容器起一个名字，可选参数，如果不给容器起名字，它就没有名字，我们管理容器的时候可以根据容器的id去管理容器，比如关闭容器，查看容器信息都可以使用id查找到这个容器
* myjava: 容器的名字
* java: 镜像的名字
* bash:启动的容器运行什么样的程序，运行的是bash命令行

另外还有一些其他参数，比如启动容器之后开启什么端口，把这个端口映射到宿主机上等

```
docker run -it --name myjava -p 9000:8080 -p 9001:8085 java bash
```

* -p: 映射端口
* 9000:8080: `9000` 代表的是宿主机的端口，`:8080` 是容器的端口，这句话的意思说把容器 `8080` 的端口映射到真实主机 `9000` 端口上；
* -p: 映射另外一组端口，容器想映射多少个端口就写多少个 `-p` 参数就可以了；后面的表示把 `8085` 端口映射到真实宿主机`9001` 端口上;

还可以把宿主机上的文件或文件夹映射到容器中,比如将来跑数据库的时候数据库存储的数据一定要保存到宿主机上的，不应该存储到容器里面，数据一定要在容器之外去保存，将来在备份和恢复的时候就非常方便。

```
docker run -it --name myjava -v /home/project:/soft --privileged java bash
```

* -v: 映射文件，如果有多个映射就使用多个 `-v`;
* /home/project:/soft: 宿主机信息，以冒号 `:` 分隔，`/home/project` 表示宿主机的目录，这句表示把宿主机的 `/home/project` 目录映射到容器中的 `/soft` 目录里面;
* --privileged: 在linux系统创建文件和读取文件都是有读写权限的，我把宿主机的目录映射到容器的目录中，`soft` 目录就可以看到真实主机的 `project` 目录中的一些东西了，如果我们想在 `soft` 目录中去创建文件和读写文件，真实的宿主机会不会给 `soft` 这样的权限呢? 后面就需要加上 `--privilged` 这样的参数，这个参数就告诉docker在运行的时候容器在操作映射目录和映射文件的时候是拥有最高权限的，读写都是可以的。

#### 实操

我们首先在 `/home` 目录中创建一个文件夹,将来把文件夹映射到容器里面。

![](https://images.notes.xuepincat.com/docker/13.png)

启动镜像，并将8080端口映射到真实主机9000上，把8085端口映射到9001端口上,还有目录的映射

```
docker run -it -p 9000:8080 -p 9001:8085 -v /home/project:/soft --privileged --name myjava docker.io/java bash
```

![](https://images.notes.xuepincat.com/docker/15.png)

回车后会发现前面的提示都变了，现在的界面是进入到了容器里面，刚才命令我们添加了 `-it` 参数，该参数就是启动一个交互的界面，这里启动的是一个java的容器，里面安装了jdk

* 检测一下java环境，输入 `javac`

![](https://images.notes.xuepincat.com/docker/16.png)

* 查看一下java版本 `java -version`

![](http://images.notes.xuepincat.com/docker/17.png)

* 查看一下映射的目录，会发现没有任何的东西。

![](https://images.notes.xuepincat.com/docker/18.png)

* 在 `soft` 目录中创建一个文件，并向里面写入内容

![](https://images.notes.xuepincat.com/docker/19.png)

* 退出当前容器

![](https://images.notes.xuepincat.com/docker/20.png)

* 进入宿主机的 `/home` 目录查看里面内容

![](https://images.notes.xuepincat.com/docker/21.png)

### 暂停容器

```
docker pause myjava
```

### 恢复容器

```
docker unpause myjava
```

### 彻底停止

```
docker stop myjava
```

### 恢复运行

```
docker start -i myjava
```

之前我们在容器的交互界面使用 `exit` 退出容器，该命令不仅是退出容器，还停止运行了，使容器进入到 `stop` 状态里面，如果要运行执行容器的话就必须使用 `start` 命令去重新启动容器。

* 重新启动刚才关闭的容器

![](https://images.notes.xuepincat.com/docker/22.png)

* 重新打开一个终端，并连接到linux上，在这里面将 `myjava` 的容器暂停一下

![](https://images.notes.xuepincat.com/docker/23.png)

* 恢复容器

![](https://images.notes.xuepincat.com/docker/24.png)

* 如果想删除容器，前提是必须彻底停止容器，然后再去删除容器

![](https://images.notes.xuepincat.com/docker/25.png)

* 查看容器

![](https://images.notes.xuepincat.com/docker/26.png)
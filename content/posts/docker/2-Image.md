+++
title = 'Docker Image'
date = 2019-05-19T09:46:13+08:00
draft = true
categories = [ "Docker" ]
tags = [ "docker" ]
+++

## 什么是Image

![](/images/docker/docker/11.png)

1.Image 是 `文件` 和 `meta data` 的集合 (root filesystem)，这里的文件其实就是 `root filesystem`
2.分层的，并且每一层都可以添加改变删除文件，称为一个新的image
Linux系统可分为内核空间和用户空间，内核空间就是 Linux Kernel，称之为 `bootfs` ，用户空间就是在 `bootfs` 上面去做的各种Linux发行版，如Ubuntu，CentOS，Debain。Docker 的 Image 也是分层的，并且每一层都可以添修改或者删除文件，使之称为一个新的 image

比如上图中可以看到在在 `bootfs` 之上建立了一些Linux发行版，称之为 `Base Image` ，这些 `Base Image` 只包含 `bootfs` ，不包含 Linux Kernel ，它可以共享主机中的 Linux Kernal ，我们可以在 Base Image 之上去添加或者删除文件， 比如在 Base Image 之上安装一些软件，如MySQL，Apache等，这样就会产生新的一层，这一层就会变成新的Image

3.不同的Image可以共享相同的layer(分层)
比如图中的Image #4和Image #2，比如Image #2里面在CentOS上安装了Apache, Image#4又在Image #2之上安装了MySQL，Image #4和Image #2共享了底层的Base Image

4.Image本身是只读的(read-only)

### 查看镜像

可以通过下面指令查看本地有哪些image

```
sudo docker image list
sudo docker image ls
```

![](/images/docker/docker/12.png)

## Image的获取

1.Build from Dockerfile

Docker 提供了一种叫 `Dockerfile` 的文件，通过这个文件可以去定义一个image，通过dockerfile又可以去构建image

2.Pull from Registry

我们可以从Registry中pull image，也可以将我们自己的image镜像push到Registry上，比如从Docker Registry中获取ubuntu20.04:

```
sudo docker pull ubuntu:20.04
```

![](/images/docker/docker/13.png)

## DIY Base Image

#### 引入

在构建之前，我们先拉取官方提供的一个叫 `hello-world` 的 image：

```
docker pull hello-world
```

![](/images/docker/docker/14.png)

这个 `hello-world` 其实就是一个 `Base Image`，这个 `Base Image` 里面包含了一个可执行文件，我们去执行这个 `image` 就是去创建一个容器 `container`，创建容器后就可以启动

```
docker run hello-world
```

这样就会去执行 `hello-world` 的 `container`，屏幕中就会打印出一段话

![](/images/docker/docker/15.png)

#### 构建 Base Image

现在自己实现创建一个类似 “hello-world” 的 `hello-docker` 的 `image`

1.创建 `hello-docker` 目录

```
mkdir hello-docker
```

2.进入目录

```
cd hello-docker
```

3.创建一个名为 `hello.c` 的文件

```
vim hello.c
```

如果没有安装vim，需要使用下面指令进行安装

```
sudo yum install -y vim 
```

内容：

```
#include <stdio.h>

int main()
{
    printf("hello docker\n");
}
```

保存退出

4.将 `hello.c` 编译成二进制可执行文件

如果没有安装 `gcc` ，可以使用下面命令进行安装 

```
sudo yum install -y gcc
sudo yum install -y glibc-static
```

开始编译，输出 `hello` 的可执行文件

```
gcc -static hello.c -o hello
```

然后就会在当前目录下生成一个 `hello` 的可执行文件：

![](/images/docker/docker/16.png)

我们可以执行下面指令直接执行这个文件：

```
./hello
```

![](/images/docker/docker/17.png)

5.通过 `Dockerfile` 将可执行文件构建打包成 `Image`,首先需要创建 `Dockerfile`

```
vim Dockerfile
```

6.键入内容

```
FROM scratch
ADD hello /
CMD ["/hello"]
```

第一行我们可以通过 `Base Image`，在 `Base Image` 之上去安装一些软件形成一个新的 `Image`，但是现在就是在写一个 `Base Image` ，所以不能从一个 `Base Image` 去安装，所以需要从 `scratch` 安装

第二行将 `hello` 添加到 `image` 的根目录中

第三行运行 command，就是运行的根目录下面的 `hello` , 这个 `hello` 就是通过第二步添加进来的

保存退出

7.构建

```
docker build -t finnley/hello-docker .
```

`-t` 指定Tag
`finnley` 是我自己取的用户名
`hello-docker` image的名称
`.` 表示在当前目录中找 `Dockerfile`

![](/images/docker/docker/18.png)

`Step` 共分3步，所以这个 `Image` 有3层

8.查看

```
docker image ls
```

![](/images/docker/docker/19.png)

可以通过下面命令查看image分层

```
docker history [docker image id]
```

![](/images/docker/docker/20.png)

可以看到这个image只要两层，第一层是添加了文件，第二层是运行执行命令，在Dockerfile中第一行是 `FROM scratch` ，表示这里不是基于任何 Base Image 的，所以它不算是一层，所以这里的 Docker Image 只有两层

9.运行

```
docker run finnley/hello-docker
```

![](/images/docker/docker/21.png)

### 构建自己的 Docker 镜像

#### `commit` 命令

```
docker container commit
```

或者 

```
docker commit
```

这条命令表示基于某个 `Image` 创建 `Container` ,然后在 `Container` 里面做一些变化，如安装软件等，然后再将已经改变的 `Container` 进行 `commit` 成一个新的 `Image`

下面在 `centos` 的基础上安装 `vim`,然后重新构建新的镜像

1. 查看 Image 列表

```
docker image ls
```

![](/images/docker/150.png)

2. 启动交互式界面运行 `centos`

```
docker run -it centos
```

![](/images/docker/151.png)

3. 接着在容器中安装软件，如 `vim`

```
yum install -y vim
```

![](/images/docker/152.png)

4. 退出,并查看容器列表

```
exit
docker container ls -a
```

![](/images/docker/153.png)


图中可以看到有一个退出状态的容器，现在目的是想将这个 `Container commit` 成一个 `Image` , 很显然，这个 `Image` 是基础 `CentOS` 的，它里面安装了 `vim` 

```
docker commit brave_germain finnley/centos-vim
```

![](/images/docker/154.png)

此时就在 `Image` 列表中看到一个 `centos-vim` 的 `Image` ，并且 `centos-vim` 的大小也比 `centos` 大一点，还有 `centos-vim` 和 `centos` 会共享很多的 layer

![](/images/docker/155.png)

图中可以看到上面 `centos` 和 下面 `centos-vim` 都具有相同的层 `9f38484d220f` ， centos-vim 只是在 `9f38484d220f` 的基础上加了一层 `69caca098a6f` ，这一层的大小是 `150M` , 这一层主要是安装了 `vim`

上面就是通过 `docker commit` 基于一个已经存在的 `Container` 去创建 `Image` , 但是这种方式去创建 `Image` 并不是很提倡，因为如果将 `Image` 发布出去，别人其实是不知道这个 `Image` 是怎么产生的，因为有可能将一些不安全的的东西放在 `Image` 里面发布出去，这样的 `Image` 就是一个不安全的 `Image` ，而是提倡使用 `Dockerfile` 创建 `Image`

###### `build` 命令

```
docker image build
```

或者

```
docker build
```

作用是 `Build an image from a Dockerfile`, 通过 `Dockerfile` 去 `build` 一个 `Image`

1. 首先将之前的 `Image` 删掉

```
docker rmi 69caca098a6f
```

2. 创建 `docker-centos-vim` 目录，并进入该目录

```
mkdir docker-centos-vim
cd docker-centos-vim
```

![](/images/docker/157.png)

3. 在 `docker-centos-vim` 目录中创建 `Dockerfile`

```
vim Dockerfile
```

键入内容

```
FROM centos
RUN yum install -y vim
```

`FROM centos` : 因为这次要在 `centos` 的基础上安装 `vim` ,所以需要 `FROM centos`, 这里的 `centos` 也就是 `BaseImage`

`RUN yum install -y vim` : 有了 `centos` 的 `Image` 之后，需要在 `Image` 里面运行 `yum install -y vim` 进行安装 `vim` ,都知道 `Image` 是只读的，在一个只读的 `Image` 里面， 是如何通过 `yum install -y vim` 去安装 `vim` 的呢? 安装 `vim` 是往里面写东西，但是 `Image` 是只读的

`:wq` 保存退出

4. 使用 `build` 命令基于 `Dockerfile` 去 `build` 一个新的 `Image`

```
docker build -t finnley/centos-vim-new .
```

`.` 表示基于当前目录下的 `Dockerfile` 进行 `build`

![](/images/docker/158.png)

图中第一步首先 `FROM centos` , 直接就会引用 `centos` 的 `9f38484d220f` 这一层，它不会去产生新的 `layer` , 第二步运行 `yum install -y vim` , 它会 `Running in 3511ee78d9c3` ，这表示它在 `build` 过程中生成新的临时的 `Container` ， `3511ee78d9c3` 就是新的临时 `Container ID` ,它会在新的临时的 `Container` 里面通过 `yum install -y vim` 去安装 `vim` 

![](/images/docker/159.png)

最后安装完成后会去移除临时的 `Container` ,并且它会基于临时的 `Container` 去 `commit` 成一个新的 `Image`

5. 查看新生成的 `Image`

```
docker image ls
```

![](/images/docker/160.png)

虽然这种生成的 `Image` 和通过 `commit` 生成的 `Image` 差不多，实际上还是有些区别的

我们生成 `Image` 提倡通过 `Dockerfile` 一步一步去完成，这样我们只需要分享 `Dockerfile` 给别人就可以了，别人通过 `Dockerfile` 就可以 `build` 出跟我一样的 `Image`
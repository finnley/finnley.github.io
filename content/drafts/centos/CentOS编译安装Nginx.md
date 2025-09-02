+++
title = 'CentOS编译安装Nginx'
date = 2019-03-10T12:47:26+08:00
draft = true
categories = [ "CentOS" ]
tags = [ "centos" ]
+++

## 编译环境准备

编译环境是指能够将源码编译成可安装的文件，这里需要系统内安装好 `gcc` 和 `gcc-c++`

* gcc
* gcc-c++

这里我使用的阿里云CentOS7.6 64位版本的系统。

首先使用远程连接工具进行进行连接，我本地物理机使用的 `Ubuntu 18.04` 自带的 `Terminal` 进行连接，如果是 `Windows` 系统可以使用 `XShell` 或者 `PuTTY` 等工具进行远程连接。

<!-- more -->

![nginx](http://images.ornnth.com/hexo/nginx/1.png)

连接上服务器后先使用 `yum update -y` 命令更新下系统。

```
yum update -y
```

![nginx](http://images.ornnth.com/hexo/nginx/2.png)

![nginx](http://images.ornnth.com/hexo/nginx/3.png)

然后使用下面命令搜索机器内是否已经安装了 `gcc` 和 `gcc-c++` ， 如果出现内容就表示已经安装

```
rpm -qa | grep gcc*
```

![nginx](http://images.ornnth.com/hexo/nginx/4.png)

如果没有出现内容就表示没有安装,例如判断系统内是否已经安装了 `Nginx`

```
rpm -qa | grep nginx
```

![nginx](http://images.ornnth.com/hexo/nginx/5.png)

显示系统内是还没有安装 `Nginx` 的。如果系统没有安装 `gcc` 和 `gcc-c++`, 可以使用下面命令进行安装 `gcc` 和 `gcc-c++`

```
yum -y install gcc-c++
```

![nginx](http://images.ornnth.com/hexo/nginx/6.png)

![nginx](http://images.ornnth.com/hexo/nginx/7.png)

现在再次使用 `rpm -qa | grep gcc*` 命令查看

```
rpm -qa | grep gcc*
```

![nginx](http://images.ornnth.com/hexo/nginx/8.png)

##　源码编译安装 `Nginx`

首先打开 [Nginx官网](http://nginx.org/en/)

![nginx](http://images.ornnth.com/hexo/nginx/9.png)

点击右边的　`download`　进入下载页面

![nginx](http://images.ornnth.com/hexo/nginx/10.png)

页面中可以看到当前最新稳定版为 `1.14.2` ,这也是我们所要编译安装的版本。

我们可以使用下面命令下载该版本

```
wget http://nginx.org/download/nginx-1.14.2.tar.gz
```

![nginx](http://images.ornnth.com/hexo/nginx/11.png)

我们可以通过 `ll` 命令查看当前目录下已经下载的安装包。

```
ll
```

![nginx](http://images.ornnth.com/hexo/nginx/12.png)

### 解压安装包

使用 `tar` 命令进行解压，然后使用 `ll` 命令查看解压后的文件
```
tar -zxvf nginx-1.14.2.tar.gz
ll
```

![nginx](http://images.ornnth.com/hexo/nginx/13.png)

![nginx](http://images.ornnth.com/hexo/nginx/14.png)

我们进入解压后的目录查看里面内容

```
cd nginx-1.14.2
ll
```

![nginx](http://images.ornnth.com/hexo/nginx/15.png)

### 配置

打开 `Nginx` 官网 ，点击右边的 `documentation`

![nginx](http://images.ornnth.com/hexo/nginx/16.png)

我们点击 `Building nginx from Sources` 使用源码构建nginx

![nginx](http://images.ornnth.com/hexo/nginx/17.png)

将页面拉到最下面，找到 `Example of parameters usage (all of this needs to be typed in one line):` 官网给的示例配置

![nginx](http://images.ornnth.com/hexo/nginx/18.png)

```
./configure
    --sbin-path=/usr/local/nginx/nginx
    --conf-path=/usr/local/nginx/nginx.conf
    --pid-path=/usr/local/nginx/nginx.pid
    --with-http_ssl_module
    --with-pcre=../pcre-8.42
    --with-zlib=../zlib-1.2.11
```

我们修改后在终端下粘贴

```
./configure \
--sbin-path=/usr/local/nginx/nginx \
--conf-path=/usr/local/nginx/nginx.conf \
--pid-path=/usr/local/nginx/nginx.pid \
--with-http_ssl_module \
--with-pcre=../pcre-8.42 \
--with-zlib=../zlib-1.2.11
```

![nginx](http://images.ornnth.com/hexo/nginx/19.png)

回车

![nginx](http://images.ornnth.com/hexo/nginx/20.png)

![nginx](http://images.ornnth.com/hexo/nginx/21.png)

在最后发现提示这样的错误 `./configure: error: SSL modules require the OpenSSL library.`

```
./configure: error: SSL modules require the OpenSSL library.
You can either do not enable the modules, or install the OpenSSL library
into the system, or build the OpenSSL library statically from the source
with nginx by using --with-openssl=<path> option.
```

![nginx](http://images.ornnth.com/hexo/nginx/22.png)

解决方法就是我们需要安装 `openssl` 和 `openssl-dev`

```
yum -y install openssl openssl-devel
```

![nginx](http://images.ornnth.com/hexo/nginx/23.png)

![nginx](http://images.ornnth.com/hexo/nginx/24.png)


### 编译

安装完成后继续执行上面的编译操作

```
./configure \
--sbin-path=/usr/local/nginx/nginx \
--conf-path=/usr/local/nginx/nginx.conf \
--pid-path=/usr/local/nginx/nginx.pid \
--with-http_ssl_module \
--with-pcre=../pcre-8.42 \
--with-zlib=../zlib-1.2.11
```

![nginx](http://images.ornnth.com/hexo/nginx/25.png)

![nginx](http://images.ornnth.com/hexo/nginx/26.png)

此时并没有发现有任何错误提示。编译完执行 `make` 操作

```
make
```

提示下面错误

```
make -f objs/Makefile
make[1]: Entering directory `/root/nginx-1.14.2'
cd ../pcre-8.42 \
&& if [ -f Makefile ]; then make distclean; fi \
&& CC="cc" CFLAGS="-O2 -fomit-frame-pointer -pipe " \
./configure --disable-shared 
/bin/sh: line 0: cd: ../pcre-8.42: No such file or directory
make[1]: *** [../pcre-8.42/Makefile] Error 1
make[1]: Leaving directory `/root/nginx-1.14.2'
make: *** [build] Error 2
```

![nginx](http://images.ornnth.com/hexo/nginx/27.png)

提示说在编译配置的时候没有找到 `../pcre-8.42` 目录，这是因为我们在配置的时候有这样的一句话 `--with-pcre=../pcre-8.42 \` 和 `--with-zlib=../zlib-1.2.11` ，我们配置的时候需要执行该目录

我们将上面打开的官网往上移动一点会看到下面的页面

![nginx](http://images.ornnth.com/hexo/nginx/28.png)

点击看色下划线的 `PCRE` 进入下面页面

![nginx](http://images.ornnth.com/hexo/nginx/29.png)

我们找到 `Download` 点击 [https://ftp.pcre.org/pub/pcre/](https://ftp.pcre.org/pub/pcre/) 进入到下面下载页面

![nginx](http://images.ornnth.com/hexo/nginx/30.png)

我们找到对应的 `pcre-8.42` 的版本复制下载链接，然后进入主目录进行下载

```
cd ~
wget https://ftp.pcre.org/pub/pcre/pcre-8.42.tar.gz
```

![nginx](http://images.ornnth.com/hexo/nginx/31.png)

然后还在 `Nginx` 官网下面，方才的 `PCRE` 下面找到 蓝色下划线的 `zlib` 字体，点击进去进入下面页面

![nginx](http://images.ornnth.com/hexo/nginx/32.png)

向下移动页面，找到下面内容

![nginx](http://images.ornnth.com/hexo/nginx/33.png)

复制下载连接进行下载

```
wget http://prdownloads.sourceforge.net/libpng/zlib-1.2.11.tar.gz?download
```

![nginx](http://images.ornnth.com/hexo/nginx/34.png)

查看下刚才下载的两个文件

```
ll
```

![nginx](http://images.ornnth.com/hexo/nginx/35.png)

分别解压 `pcre-8.42.tar.gz` 和 `zlib-1.2.11.tar.gz` 文件

```
tar -zxvf pcre-8.42.tar.gz
tar -zxvf zlib-1.2.11.tar.gz
```

![nginx](http://images.ornnth.com/hexo/nginx/36.png)

下面进入 `nginx` 的解压目录，重新执行配置编译操作

```
cd nginx-1.14.2
./configure \
--sbin-path=/usr/local/nginx/nginx \
--conf-path=/usr/local/nginx/nginx.conf \
--pid-path=/usr/local/nginx/nginx.pid \
--with-http_ssl_module \
--with-pcre=../pcre-8.42 \
--with-zlib=../zlib-1.2.11
```

![nginx](http://images.ornnth.com/hexo/nginx/37.png)

![nginx](http://images.ornnth.com/hexo/nginx/38.png)

然后进行 `make` 操作，在做每一个操作的时候要确定每一步都是没有问题的。

执行 `make` 操作

```
make
```

![nginx](http://images.ornnth.com/hexo/nginx/39.png)

![nginx](http://images.ornnth.com/hexo/nginx/40.png)

此时没有出错的提示信息，代表编译完成。

### 安装

编译完成后执行 `make install` 操作

```
make install
```

![nginx](http://images.ornnth.com/hexo/nginx/41.png)

![nginx](http://images.ornnth.com/hexo/nginx/42.png)

此时 `Nginx` 已经安装完成。

默认情况下，`Nginx` 会被默认安装到 `/usr/local` 下的 `nginx` 目录下

![nginx](http://images.ornnth.com/hexo/nginx/43.png)

此时我们打开浏览器，在地址栏中输入服务器IP，回车后发现页面如下所示：

![nginx](http://images.ornnth.com/hexo/nginx/44.png)

进入 `nginx` 安装目录 `/usr/local/nginx`， 输入 `./nginx` 命令来启动 `nginx`

```
./nginx
```

另外可以可以通过下面命令来查看 `nginx` 系统进程，用此来判断是否启动成功。

```
ps -aux | grep nginx
```

![nginx](http://images.ornnth.com/hexo/nginx/45.png)

现在刷新一下刚才打开的网页，此时就可以打开了。

![nginx](http://images.ornnth.com/hexo/nginx/46.png)

进入 `nginx` 安装目录中的 `html` 目录，编写的一个测试文件 `test.html` , 在文件中输入 `Hello Nginx!` 的内容，然后在浏览器地址栏中输入 `IP/test.html`

![nginx](http://images.ornnth.com/hexo/nginx/47.png)




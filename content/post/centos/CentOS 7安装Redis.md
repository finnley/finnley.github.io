+++
title = 'CentOS 7安装Redis'
date = 2018-07-26T10:24:47+08:00
draft = true
categories = [ "Redis" ]
tags = [ "redis" ]
+++

### 下载 `Redis`
我们可以打开官网，找到 `Download` ，选择 `Stable` 稳定版 进行下载

![](/images/redis/20180726/1.png)

进入linxu终端，下载 `redis`

<!-- more -->

```
wget http://download.redis.io/releases/redis-4.0.10.tar.gz
```

![](/images/redis/20180726/2.png)

### 解压

```
tar zxvf redis-4.0.10.tar.gz 
```

完成解压后出现下面的目录

![](/images/redis/20180726/3.png)

### 安装

进入解压后的目录

使用 `make` 命令，并指定安装目录，这里指定安装到 `/usr/local/redis` 目录下

```
cd redis-4.0.10
make PREFIX=/usr/local/redis install
```

进入 `/usr/local/redis` 中，查看相关内容

![](/images/redis/20180726/4.png)

里面就是我们的一些脚本，如启动脚本

### 配置启动redis

我们回到解压后的目录中，里面有个 `redis.conf` 的配置文件，它就是我们的redis配置文件，当然现在它是在安装包中，肯定是不能用的

![](/images/redis/20180726/5.png)

想要使用我们可以将它复制到我们的redis安装路径下

首先我们在redis安装目录下创建 `etc` 目录作为存放配置文件的目录,然后把 `redis.conf` 文件复制到etc目录

```
mkdir -p /usr/local/redis/etc
cp redis.conf /usr/local/redis/etc/
```

![](/images/redis/20180726/6.png)

接下来开始制作启动脚本

还是在解压目录中，进入 `utils` 工具目录，它里面有个 `redis_init_script` 的启动脚本，

![](/images/redis/20180726/7.png)

修改该启动配置文件

```
vi redis_init_script
```

找到下面代码进行修改,将配置路径修改我安装的redis路径

```
EXEC=/usr/local/bin/redis-server
CLIEXEC=/usr/local/bin/redis-cli
```

修改后

```
EXEC=/usr/local/redis/bin/redis-server
CLIEXEC=/usr/local/redis/bin/redis-cli
```

![](/images/redis/20180726/8.png)

接着做一个软链，将 `/usr/local/redis/etc/redis.conf` 软链到 `/etc/redis` 目录中，如果 `/etc` 目录下没有redis 目录就先创建一个

```
mkdir /etc/redis
ln -s /usr/local/redis/etc/redis.conf /etc/redis/6379.conf
```

![](/images/redis/20180726/9.png)

这样我们就有了配置文件了，但是现在我们的启动脚本还是解压后的目录下，此时需要将这个启动脚本复制到 `/etc/init.d` 目录下

```
cp redis_init_script /etc/init.d/redis
```

![](/images/redis/20180726/10.png)

此时已经有了redis的启动命令

开始启动

```
/etc/init.d/redis start
```

![](/images/redis/20180726/11.png)

### 后台运行redis

但是现在发现redis占据的我们的终端

```
vi /etc/redis/6379.conf
```

找到 `daemonize no`,将 `no` 改为 `yes`

然后再去启动redis

```
/etc/init.d/redis start
```

![](/images/redis/20180726/12.png)

查看是否启动成功

```
netstat -tunpl | grep 6379
```

![](/images/redis/20180726/13.png)

### 设置开机自启动

```
vi /etc/init.d/redis
```

在开头添加下面代码

```
#chkconfig:2345 80 90
```

![](/images/redis/20180726/14.png)

然后终端执行

```
chkconfig --add redis
chkconfig redis on
```

![](/images/redis/20180726/15.png)

到这里我们redis已经安装完成

可以用过 `service` 命令来进行停止和启动

```
service redis stop
service redis start
```

![](/images/redis/20180726/16.png)

### 测试

```
/usr/local/redis/bin/redis-cli
```

![](/images/redis/20180726/17.png)
</div>
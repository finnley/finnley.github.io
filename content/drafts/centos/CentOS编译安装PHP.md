+++
title = 'CentOS编译安装Nginx'
date = 2019-03-10T18:52:16+08:00
draft = true
categories = [ "CentOS" ]
tags = [ "centos" ]
+++

## 下载 `PHP` 源码

这里我使用的阿里云CentOS7.6 64位版本的系统。

首先使用远程连接工具进行进行连接，我本地物理机使用的 `Ubuntu 18.04` 自带的 `Terminal` 进行连接，如果是 `Windows` 系统可以使用 `XShell` 或者 `PuTTY` 等工具进行远程连接。

<!-- more -->

![nginx](http://images.ornnth.com/hexo/nginx/1.png)

打开 [PHP官网](http://php.net/)

![php](/images/php/5.png)

找到右上角的 `Download` 区域，选择当前最新版本 `7.3.3`

![php](/images/php/6.png)

这里选择第一个 `php-7.3.3.tar.bz2` 格式的安装包，进入下面页面，随便找一个站点并复制下载链接进行下载

![php](/images/php/7.png)

```
wget -O php-7.3.3.tar.bz2 http://php.net/get/php-7.3.3.tar.bz2/from/this/mirror
```

![php](/images/php/8.png)

下载结束会发现有一个 `php-7.3.3.tar.bz2
` 的文件

![php](/images/php/9.png)

将 `php-7.3.3.tar.bz2
` 文件进行解压

```
tar -jxvf php-7.3.3.tar.bz2
```

但是在解压的时候提示下面错误

![php](/images/php/10.png)

这是因为么有安装 `bzip`， 使用下面指令进行安装 

```
yum -y install bzip2
```

然后再进行解压操作

```
tar -jxvf php-7.3.3.tar.bz2
```

解压后会多出 `php-7.3.3` 的文件夹

![php](/images/php/11.png)

### 编译

打开官网，找到上面 `Documentation` 导航到下面页面

![php](/images/php/12.png)

![php](/images/php/12.png)

选择 `Chinese (Simplified)` 简体中文，进入下面页面

![php](/images/php/13.png)

找到 `安装与配置` 下面的 `Unix 系统下的安装` ，进入下面页面

![php](/images/php/14.png)

找到对应的 `Nginx` 版本 `Unix 系统下的 Nginx 1.4.x` ，进入下面页面

![php](/images/php/15.png)

我们就可以根据官网指示来进行安装。

### 编译

进入源码目录

```
cd php-7.3.3
```

在编译之前使用下面命令查看编译参数

```
./configure --help
```

开始编译


```
./configure --enable-fpm --with-mysqli --with-pdo-mysql
```

编译结束后会发现出现下面错误提示 `configure: error: libxml2 not found. Please check your libxml2 installation.
`

这表示我们没有安装 `libxml2-devel` , 可以使用下面命令进行安装

```
yum -y install libxml2-devel
```

![php](/images/php/16.png)

![php](/images/php/17.png)

安装完后重新执行配置操作

```
./configure --enable-fpm --with-mysqli --with-pdo-mysql
```

![php](/images/php/18.png)

当看到屏幕中出现 `Thank you for using PHP.
` 这句话的时候表示编译成功。

### 编译

安装执行 `make` 指令

```
make
```

![php](/images/php/19.png)

### 安装

编译结束后执行 `make install` 操作

```
make install
```

![php](/images/php/20.png)

根据官网步骤，创建配置文件，并将其复制到正确的位置。

```
cp php.ini-development /usr/local/php/php.ini
cp /usr/local/etc/php-fpm.conf.default /usr/local/etc/php-fpm.conf
cp sapi/fpm/php-fpm /usr/local/bin
```

需要着重提醒的是，如果文件不存在，则阻止 Nginx 将请求发送到后端的 PHP-FPM 模块， 以避免遭受恶意脚本注入的攻击。

将 php.ini 文件中的配置项 cgi.fix_pathinfo 设置为 0 。

打开 php.ini:

```
vim /usr/local/php/php.ini
```

定位到 cgi.fix_pathinfo= 并将其修改为如下所示：

```
cgi.fix_pathinfo=0
```

然后启动 php-fpm 服务：

```
/usr/local/bin/php-fpm
```

启动的时候出现了下面的问题

```
[root@iZuf6hrgs866a8qloc5ijsZ php-7.3.3]# /usr/local/bin/php-fpm
[11-Mar-2019 22:11:59] ERROR: Unable to globalize '/usr/local/NONE/etc/php-fpm.d/*.conf' (ret=2) from /usr/local/etc/php-fpm.conf at line 143.
[11-Mar-2019 22:11:59] ERROR: failed to load configuration file '/usr/local/etc/php-fpm.conf'
[11-Mar-2019 22:11:59] ERROR: FPM initialization failed
[root@iZuf6hrgs866a8qloc5ijsZ php-7.3.3]#
```

![php](/images/php/21.png)

解决方法：

编辑 `/usr/local/etc/php-fpm.conf` 文件， 找到 `include=NONE/etc/php-fpm.d/*.conf` ，将其改为 `include=/usr/local/etc/php-fpm.d/*.conf`

修改前：

![php](/images/php/22.png)

修改后：

![php](/images/php/23.png)

如果没有 `/usr/local/etc/php-fpm.d` 目录，就自己创建一个。

然后再去执行 `/usr/local/bin/php-fpm` 命令来启动 `php-fpm`

```
/usr/local/bin/php-fpm
```

但是现在又提示了下面的信息

```
[root@iZuf6hrgs866a8qloc5ijsZ php-7.3.3]# /usr/local/bin/php-fpm
[11-Mar-2019 22:23:45] WARNING: Nothing matches the include pattern '/usr/local/etc/php-fpm.d/*.conf' from /usr/local/etc/php-fpm.conf at line 143.
[11-Mar-2019 22:23:45] ERROR: No pool defined. at least one pool section must be specified in config file
[11-Mar-2019 22:23:45] ERROR: failed to post process the configuration
[11-Mar-2019 22:23:45] ERROR: FPM initialization failed
[root@iZuf6hrgs866a8qloc5ijsZ php-7.3.3]# 
```

![php](/images/php/24.png)

解决方法：

```
cp /usr/local/etc/php-fpm.d/www.conf.default /usr/local/etc/php-fpm.d/www.conf

```

然后再去执行 `/usr/local/bin/php-fpm` 命令来启动 `php-fpm`

```
/usr/local/bin/php-fpm
```

运行之后查看php-fpm服务运行状态

```
ps -e | grep php-fpm
```

![php](/images/php/25.png)

配置 Nginx 使其支持 PHP 应用：

```
cd /usr/local/nginx
vim nginx.conf
```

![php](/images/php/26.png)

修改默认的 location 块，使其支持 .php 文件：

```
location / {
    root   html;
    index  index.php index.html index.htm;
}
```

下一步配置来保证对于 .php 文件的请求将被传送到后端的 PHP-FPM 模块， 取消默认的 PHP 配置块的注释，并修改为下面的内容：

```
location ~* \.php$ {
    fastcgi_index   index.php;
    fastcgi_pass    127.0.0.1:9000;
    include         fastcgi_params;
    fastcgi_param   SCRIPT_FILENAME    $document_root$fastcgi_script_name;
    fastcgi_param   SCRIPT_NAME        $fastcgi_script_name;
}
```

重启 Nginx。

```
./nginx
```

但是现在提示下面信息，其实在上一章中nginx已经启动了

![php](/images/php/27.png)

通过下面命令查看

```
ps -e | grep nginx
```

![php](/images/php/28.png)

下面使用nginx提供的动态加载其配置

```
./nginx -s reload
```

![php](/images/php/29.png)

现在进入站点目录 

```
cd html
```

创建一个 `index.php` 的文件，内容为下面内容

``` php
<?php
	phpinfo();
```

浏览器中访问该文件 `IP/index.php`

![php](/images/php/30.png)
</div>
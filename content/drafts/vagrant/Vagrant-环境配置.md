+++
title = 'Vagrant环境配置'
date = 2018-12-16T22:17:11+08:00
draft = true
categories = [ "Vagrant" ]
tags = [ "vagrant" ]
+++

## 搭建LANMP

* 进入虚拟机

```
vagrant ssh
```

![](https://images.notes.xuepincat.com/vagrant/6.png)

* 备份源

```
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
```

* 编辑源

```
sudo vi /etc/apt/sources.list
```

* 清空全部内容

```
:1,$d
#修改为阿里源
deb http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse
```

* 更新源

```
sudo apt-get update
```

## 安装 Apache 和 Nginx

* 查看nginx是否安装

```
apt-cache search nginx
```

* 安装nginx

```
sudo apt-get install nginx
```

* 查看nginx版本

```
nginx -v
```

* nginxc测试

```
curl -I 'http://127.0.0.1'
```

![](https://images.notes.xuepincat.com/vagrant/7.png)

* 安装apache

```
sudo apt-get install apache2
```

* 查看apache版本

```
apache2 -v
```

* apache测试 由于nginx和apche对占用同样的端口，这里先讲nginx停止

```
sudo /etc/init.d/nginx stop
```

```
curl -I 'http://127.0.0.1'
```

* 启动apache

```
sudo /etc/init.d/apache2 start
```

```
curl -I 'http://127.0.0.1'
```

![](https://images.notes.xuepincat.com/vagrant/8.png)

## 安装MySQL

```
sudo apt install mysql-server mysql-client
#测试是否安装
mysql -u root -p
>Enter password:
```

![](https://images.notes.xuepincat.com/vagrant/9.png)

## 安装PHP

```
#查看php命令是否存在
php

# 安装php5-cli
sudo apt-get install php5-cli

#查看php版本
php -v
```

![](https://images.notes.xuepincat.com/vagrant/10.png)

![](https://images.notes.xuepincat.com/vagrant/11.png)

## 安装PHP扩展

```
sudo apt-get install php5-mcrypt php5-mysql php5-gd
```

## 安装Apache PHP模块

```
sudo apt-get install libapache2-mod-php5
```

## 安装Nginx PHP模块

```
sudo apt-get install php5-cgi php5-fpm
```

查看apache是否启动状态

```
ps -ef | grep apache
```

查看nginx是否启动状态

```
ps -ef | grep nginx
```

![](https://images.notes.xuepincat.com/vagrant/12.png)

启动nginx,此时发现nginx并没有启动成功，是因为apache和nginx默认监听的端口都是80端口

![](https://images.notes.xuepincat.com/vagrant/13.png)

为了解决上面情况讲apache设置的监听端口为8888 nginx监听的端口为80

## 修改Apache2配置文件

```
cd /etc/apache2/
sudo vi ports.conf 
```

讲默认的80改为8888

![](https://images.notes.xuepincat.com/vagrant/14.png)

重启apache2

```
sudo /etc/init.d/apache2 restart
```

然后再去启动nginx

```
sudo /etc/init.d/nginx start
```

![](https://images.notes.xuepincat.com/vagrant/15.png)

## 配置使浏览器能够访问

```
#退出
exit
#挂起虚拟机
vagrant suspend
```

![](https://images.notes.xuepincat.com/vagrant/16.png)

点击VirtualBox 【Settings】 -> 【Network】 -> 【Advanced】 -> 【Port Forwarding】 

修改端口转发配置如下

![](https://images.notes.xuepincat.com/vagrant/17.png)

启动虚拟机

```
vagrant up
vagrant ssh
```

浏览器访问测试

![](https://images.notes.xuepincat.com/vagrant/18.png)
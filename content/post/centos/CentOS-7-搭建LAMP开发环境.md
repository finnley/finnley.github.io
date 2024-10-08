+++
title = 'CentOS搭建LAMP开发环境'
date = 2017-12-21T23:28:01+08:00
draft = true
categories = [ "CentOS" ]
tags = [ "centos" ]
+++


# 系统环境

腾讯云 `CentOS 7`

# 安装 Apache

首先将系统软件包更新到最新版本

``` bash
# yum -y update
```

安装 Apache HTTP server

``` bash
# yum -y install httpd
```

开启 `httpd` 服务，设置开机自启动，查看 `httpd` 服务状态

``` bash
＃ systemctl start httpd
＃ systemctl enable httpd
＃ systemctl status httpd
```

![](/images/20171221/1.png)

测试。在浏览器地址栏中通过ip去访问

![](/images/20171221/7.png)

# 安装MySQL

添加MySQL Yum 仓库，我们从 [MySQL Yum仓库](http://dev.mysql.com/downloads/repo/yum/) 下载适合电脑版本的 rpm包。

先查看系统版本

``` bash
# uname -a
```

![](/images/20171221/8.png)

我们需要安装和该系统匹配的 MySQL版本，演示系统为 `el7`，所以选择 MySQL 版本是第一个。

![](/images/20171221/9.png)

下载

``` bash
# wget https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
# rpm -Uvh mysql57-community-release-el7-11.noarch.rpm
```

![](/images/20171221/10.png)

安装MySQL（官网最新版本）

``` bash
# yum -y install mysql-community-server
```

启动MySQL

``` bash
# service mysqld start
```

![](/images/20171221/11.png)

或者

``` bash
# systemctl start mysqld
```

![](/images/20171221/12.png)

查看MySQL服务状态

``` bash
# service mysqld status
```

![](/images/20171221/13.png)

或者

``` bash
# systemctl status mysqld.service
```

![](/images/20171221/14.png)

设置开机启动

``` bash
# systemctl enable mysqld
# systemctl daemon-reload
```

![](/images/20171221/15.png)

**在MySQL的安装过程中完成了以下一些事情：**

> 1. 安装了mysql服务
> 2. 生成SSL证书文件并存放在data目录
> 3. 安装有效性密码验证插件并启用
> 4. 本地超级用户root被创建，root用户的密码在日志文件中，使用下面的命令查看密码

``` bash
# grep 'temporary password' /var/log/mysqld.log
```

![](/images/20171221/16.png)

连接MySQL

![](/images/20171221/17.png)

修改root登录密码

``` sql
mysql> alter user 'root'@'localhost' identified by 'Your New Password';
```

或者

``` sql
mysql> set password for 'root'@'localhost'=password('Your New Password');
```

*PS:*

>  MySQL5.7默认安装了密码安全检查插件（validate_password），默认密码检查策略要求密码必须包含：大小写字母、数字和特殊符号，并且长度不能少于8位。否则会提示`ERROR 1819 (HY000): Your password does not satisfy the current policy requirements`错误。

![](/images/20171221/18.png)

我这边解决方式是采用禁用密码策略。

在`/etc/my.cnf`文件添加`validate_password_policy`配置，指定密码策略。

> 选择`0（LOW），1（MEDIUM），2（STRONG）`其中一种，选择2需要提供密码字典文件,validate_password_policy=0。如果不需要密码策略，添加my.cnf文件中添加如下配置禁用即可：`validate_password = off`

![](/images/20171221/19.png)

重新启动mysql服务使配置生效：

``` bash
# systemctl restart mysqld
```

此时再去修改密码

![](/images/20171221/20.png)

## 远程连接MySQL

默认只允许root帐户在本地登录，如果要在其它机器上连接MySQL，必须修改root允许远程连接，或者添加一个允许远程连接的帐户

打开Navicat，输入连接ip,会出现下面错误

![](/images/20171221/21.png)

## 解决方法

终端连接MySQL

``` bash
# grant all privileges on *.* to 'root'@'%' identified by '123';
```

![](/images/20171221/22.png)

再次远程连接

![](/images/20171221/23.png)

## 配置默认编码为utf8

修改/etc/my.cnf配置文件，在 `[mysqld]` 下添加编码配置，如下所示：

``` vim
[mysqld]
character_set_server=utf8
init_connect='SET NAMES utf8'
```

![](/images/20171221/24.png)

# 安装PHP7

安装 Yum仓库

``` bash
# yum -y install epel-release
```

![](/images/20171221/25.png)

执行

``` bash
# rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
```

![](/images/20171221/26.png)

安装PHP 7.2(最新版本)

``` bash
# yum --enablerepo=remi-php72 -y install php
```

查看PHP版本

``` bash
# php -v
```

![](/images/20171221/27.png)

安装PHP模块

``` bash
# yum --enablerepo=remi-php72 -y install php-fpm php-xml php-soap php-xmlrpc php-mbstring php-json php-gd php-mcrypt
```

启动php-fpm

``` bash
# systemctl start php-fpm
# systemctl restart httpd
```
![](/images/20171221/28.png)


检测

在/var/www/html目录下创建info.php文件

``` bash
# vi /var/www/html/info.php
```

代码块

``` php
<?php
  phpinfo();
```

保存退出后在浏览器中输入网址`{IP}/info.php`

![](/images/20171221/29.png)

到这里CentOS下LAMP开发搭建完成。

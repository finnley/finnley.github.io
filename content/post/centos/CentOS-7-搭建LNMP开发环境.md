+++
title = 'CentOS-7-搭建LNMP开发环境'
date = 2018-09-22T11:06:10+08:00
draft = false
categories = [ "CentOS" ]
tags = [ "centos" ]
+++

## 更新系统

### 更新系统

```
yum update -y
```

### 查看系统版本

```
cat /etc/centos-release
CentOS Linux release 7.5.1804 (Core) 
```

<!-- more -->

## 安装 `Nginx`

### 配置 `nginx` 官方源

安装 `stable`版

```
nano /etc/yum.repos.d/nginx.repo
```

复制粘贴下面代码

```
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/7/$basearch/
gpgcheck=0
enabled=1
```

`Ctrl + X` 然后 `y` 保存退出

更新

```
yum update
```

### 安装 `nginx`

```
yum install -y nginx
```

### 启动服务并设置开机启动

```
systemctl start nginx
systemctl enable nginx
```

## 安装 `MariaDB`

### 配置 `MariaDB` 官方源

首先需要定制 [<span style="color:#FF1493;">MariaDB的官方源</span>](https://downloads.mariadb.org/mariadb/repositories/#mirror=neusoft), 选择合适的系统，系统版本，及 `MariaDB` 版本（最新是10.3）。

[<span style="color:#FF1493;">配置源方法</span>](https://mariadb.com/kb/en/library/yum/)

```
nano /etc/yum.repos.d/MariaDB.repo
```

复制粘贴下面内容

```
# MariaDB 10.3 CentOS repository list - created 2018-09-22 03:26 UTC
# http://downloads.mariadb.org/mariadb/repositories/
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.3/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
```

`Ctrl + X` 然后 `y` 保存退出

更新

```
yum update
```

### 安装 `MariaDB`

```
sudo yum install MariaDB-server MariaDB-client -y
```

### 启动服务并设置开机启动

```
systemctl start mariadb
systemctl enable mariadb
```

### 配置

MariaDB对MySQL的命令具有良好的兼容性。
此步主要是MariaDB的安全设置：

```
mysql_secure_installation
```

```

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none): 
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

Set root password? [Y/n] y
New password: 
Re-enter new password: 
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] y
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
[root@localhost ~]# 
```

登录

```
[root@localhost ~]# mysql -u root -p
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 16
Server version: 10.3.9-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> select version();
+----------------+
| version()      |
+----------------+
| 10.3.9-MariaDB |
+----------------+
1 row in set (0.000 sec)

MariaDB [(none)]> 
```

## 安装 `PHP`

### 配置 `php` 官方源

 [<span style="color:#FF1493;">webtatic</span>](https://webtatic.com/)源更新较快，且其命名有自己的特色方式，可以避免与其他源的某些冲突：


```
yum install epel-release
rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
```

更新

```
yum update
```

安装 `php`

```
yum install -y php72w-fpm php72w-devel php72w-gd php72w-mysqlnd php72w-pdo php72w-mbstring php72w-xmlrpc php72w-devel
```

### 启动服务并设置开机启动

```
systemctl start nginx
systemctl enable nginx
```

### 配置PHP

```
vi /etc/php.ini
```

找到

```
;cgi.fix_pathinfo=1
```

去掉注释，并将1改成0

```
cgi.fix_pathinfo=0
```

保存退出。

## 配置 `Nginx`

### 配置nginx，以支持PHP

```
cp /etc/nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf.bak
vi /etc/nginx/conf.d/default.conf 
```

修改前的默认配置是这样的：

```
server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}
```

修改如下区块，取消注释，并修改部分内容：

```
 	# pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    location ~ \.php$ {
        root           /usr/share/nginx/html;
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }
```

### 重启nginx和PHP

```
systemctl restart nginx
systemctl restart php-fpm
```

## 测试PHP是否正常运行

```
vi /usr/share/nginx/html/phpinfo.php
```

写入如下代码，并保存

```
<?php
	phpinfo();

```

再次访问你的主机地址或域名：

http://1.2.3.4/phpinfo.php

或者

http://www.urwp.com/phpinfo.php

可见到php相关信息，说明PHP和nginx已经配合工作了。

此时LNMP网络服务环境就已初步搭建了。
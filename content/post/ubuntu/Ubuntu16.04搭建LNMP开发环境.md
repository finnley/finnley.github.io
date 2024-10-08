+++
title = 'Ubuntu16.04搭建LNMP开发环境'
date = 2017-06-14T11:10:36+08:00
draft = true
categories = [ "Ubuntu" ]
tags = [ "ubuntu" ]
+++

# 安装Nginx

当前最新版的 `nginx` 稳定版是 `nginx-1.12.0` 

![](/images/ubuntu/lnmp/1.png)

1. 加入 Nginx 的 PPA

``` bash
$ sudo add-apt-repository ppa:nginx/stable
```

2. 更新系统

``` bash
$ sudo apt-get update
```

3. 安装

``` bash
$ sudo apt-get install nginx
```

![](/images/ubuntu/lnmp/4.png)

4. 查看版本

``` bash
$ nginx -v
```

5. 测试

打开浏览器，在网址栏中输入`localhost` 或者`127.0.0.1`，回车之后看下是否会出现下面页面，出现了下面页面就表示安装完成。

![](/images/ubuntu/lnmp/5.png)

6. 防火墙设置

``` bash
$ sudo ufw allow 'Nginx HTTP'
```

![](/images/ubuntu/lnmp/6.png)

# 安装MySQL

1. 安装

``` bash
$ sudo apt-get -y install mysql-server
```

2. 输入密码

![](/images/ubuntu/lnmp/7.png)

3. 输入确认密码，然后就是耐心等待了，时间也不是很长

![](/images/ubuntu/lnmp/8.png)

4. 安全设置

``` bash
$ mysql_secure_installation
```

![](/images/ubuntu/lnmp/12.png)

5. 登录测试

![](/images/ubuntu/lnmp/13.png)

# 安装PHP7.1

1. 安装最新版的php7.1

``` bash
$ sudo apt-get install python-software-properties
$ sudo add-apt-repository ppa:ondrej/php
$ sudo apt-get update
$ sudo apt-get install php7.1-fpm php7.1-mysql
```

![](/images/ubuntu/lnmp/14.png)
![](/images/ubuntu/lnmp/15.png)
![](/images/ubuntu/lnmp/16.png)

2. php配置

打开php.ini文件

``` bash
$ sudo gedit /etc/php/7.1/fpm/php.ini
```

找到 `;cgi.fix_pathinfo=1`，将前面的分号 `;` 去掉，并将 `1` 改为 `0` 

修改前：

``` bash
;cgi.fix_pathinfo=1
```

修改后：

``` bash
cgi.fix_pathinfo=0
```

重启 `php7.1-fpm`，因为上面修改了配置文件，所以需要重启才能使修改的内容生效

``` bash
$ sudo systemctl restart php7.1-fpm
```

3. nginx配置

首先备份一下文件

``` bash
$ sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/default.bak
```

修改 `default` 文件

``` bash
$ sudo gedit /etc/nginx/sites-available/default
```

将default里面的内容修改如下：

``` bash
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /var/www/html;
    index index.php index.html index.htm index.nginx-debian.html;

    server_name server_domain_or_IP;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php7.1-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

检验一下nginx是否配置成功

``` bash
$ sudo nginx -t
```

![](/images/ubuntu/lnmp/18.png)

修改完配置文件需要重新加载nginx

``` bash
$ sudo systemctl reload nginx
```

# 测试

在 `/var/www/html` 新建 `info.php` 文件

``` bash
$ sudo nano /var/www/html/info.php
```

在文件中输入下面语句

``` bash
<?php
    phpinfo();
```

然后打开浏览器，在网址栏中输入 `http://localhost/info.php` 进行测试，当出现下面界面时就表示LNMP环境已经搭建完成。

![](/images/ubuntu/lnmp/22.png)

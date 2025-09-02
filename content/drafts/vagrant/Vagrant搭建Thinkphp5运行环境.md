+++
title = 'Vagrant搭建Thinkphp5运行环境'
date = 2018-12-22T11:30:32+08:00
draft = true
categories = [ "Vagrant" ]
tags = [ "vagrant" ]
+++

## 下载TP5

进入/home/wwww目录,创建vagrant的目录

```
cd /home/www
sudo mkdir vagrant
```

先安装下必要的软件，如git,composer等

```
sudo apt-get update
sudo apt-get install git
```

```
php -r "copy('https://install.phpcomposer.com/installer', 'composer-setup.php');"
php composer-setup.php
php -r "unlink('composer-setup.php');"
sudo mv composer.phar /usr/local/bin/composer
composer config -g repo.packagist composer https://packagist.phpcomposer.com
```

git克隆tp5框架代码

```
git clone https://github.com/top-think/think tp5
```

<div align=center>
![vagrant](/images/vagrant/36.png)
</div>

进入tp5目录克隆核心代码

```
cd tp5
git clone https://github.com/top-think/framework thinkphp
```

## 配置nginx

```
cd /etc/nginx/conf.d

sudo touch tp5.conf
sudo vim tp5.conf
```

```
server {
        server_name vagrant.tp5.test;
        root /home/www/vagrant/tp5/public;

        index index.php index.html;

        location / {
                if(-f $request_filename){
                        break;
                }
                if(!-e $request_filename) {
                        rewrite ^/(.*)$ /index.php/$1 last;
                        break;
                }
        }

        location ~ \.php {
                include fastcgi_params;
                fastcgi_pass 127.0.0.1:9000;
                try_files $uri = 404;
        }
}
```

```
sudo service nginx restart
```

在host中绑定刚才配置的域名

```
sudo vim /etc/hosts

192.168.199.101         vagrant.tp5.test;
```

<div align=center>
![vagrant](/images/vagrant/37.png)
</div>

此时到浏览器中输入域名进行访问，但是会发现出错了

打开 `/etc/nginx/nginx.conf` 文件找到日志所在的位置

<div align=center>
![vagrant](/images/vagrant/38.png)
</div>

我们查看日志内容发现里面提示下面错误信息，权限问题

<div align=center>
![vagrant](/images/vagrant/39.png)
</div>

解决方法授予权限

```
sudo chmod 777 /var/log/nginx/access.log 
sudo chmod 777 /var/log/nginx/error.log 
```

<div align=center>
![vagrant](/images/vagrant/40.png)
</div>

然后再刷新浏览器，发现页面还是没有任何反应，我们继续查看刚才的日志信息

```
cd /etc/php5/fpm/pool.d
sudo vim www.conf
```
搜索 `listen` 关键字

找到下面这句话

```
listen = /var/run/php5-fpm.sock
```

将其注释修改为下面这样

```
;listen = /var/run/php5-fpm.sock
listen = 127.0.0.1:9000
```

<div align=center>
![vagrant](/images/vagrant/41.png)
</div>

然后重启fpm服务

```
sudo /etc/init.d/php5-fpm restart

```
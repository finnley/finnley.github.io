+++
title = 'Docker搭建php+nginx环境'
date = 2024-10-24T09:26:10+08:00
draft = true
+++

# Docker 方式

1、启动 Alpine 容器

```bash
docker run -it --rm --name my-alpine-container -p 8080:80 alpine:latest /bin/sh
```

2、更新索引包并安装常用工具


```bash
apk update
apk add bash curl nano vim git
```

3、安装 Nginx 和 PHP

```bash
apk add nginx php php-fpm php-cli php-mbstring php-openssl php-phar php-curl php-xml php-xmlwriter php-json php-dom php-tokenizer
```

4、配置 PHP-FPM

编辑 `/etc/php83/php-fpm.d/www.conf`，确保 `listen = 127.0.0.1:9000`。
启动 PHP-FPM 服务：

```bash
php-fpm83
```

在查看php进程：
```bash
/ # ps -ef | grep php
   29 root      0:00 {php-fpm83} php-fpm: master process (/etc/php83/php-fpm.conf)
   30 nobody    0:00 {php-fpm83} php-fpm: pool www
   31 nobody    0:00 {php-fpm83} php-fpm: pool www
   33 root      0:00 grep php
/ #
```

5、配置 Nginx

在 /etc/nginx/http.d/ 目录下创建项目独立的配置文件 test.conf：

```bash
cat > /etc/nginx/http.d/test.conf <<'EOF'
server {
    listen 80;
    server_name mydomain.com;  # 替换为你的域名

    root /var/www/myproject;
    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }

    location ~ /\.ht {
        deny all;
    }

    error_log /var/log/nginx/myproject_error.log;
    access_log /var/log/nginx/myproject_access.log;
}
EOF
```

6、创建 Web 目录和示例文件

```bash
mkdir -p /var/www/myproject
echo "<?php phpinfo(); ?>" > /var/www/myproject/index.php
```

7、启动 Nginx

```bash
nginx
```

查看nginx进程：

```bash
~ # ps -ef | grep nginx
   38 root      0:00 nginx: master process nginx
   39 nginx     0:00 nginx: worker process
   40 nginx     0:00 nginx: worker process
   41 nginx     0:00 nginx: worker process
   42 nginx     0:00 nginx: worker process
   46 root      0:00 grep nginx
~ #
```

8、测试访问

在宿主机的 `/etc/hosts` 中添加条目以支持虚拟域名：

```bash
127.0.0.1 mydomain.com
```

然后在浏览器中通过 http://mydomain.com:8080 访问。

# Docker Compose 方式

将上述流程集成到 Docker Compose。

1、创建 Dockerfile

通过创建 Dockerfile 来定义如何构建包含 Nginx 和 PHP 的自定义镜像

```Dockerfile
# 使用官方的 Alpine 镜像作为基础
FROM alpine:latest

# 更新包索引并安装 Nginx 和 PHP
RUN apk update && \
    apk add nginx php php-fpm php-cli && \
    mkdir -p /run/nginx

# 配置 PHP-FPM（这个默认支持）
# RUN sed -i 's/;listen = 127.0.0.1:9000/listen = 127.0.0.1:9000/' /etc/php83/php-fpm.d/www.conf

# 创建 Nginx 项目配置目录并添加配置文件
RUN mkdir -p /etc/nginx/http.d && \
    mkdir -p /var/www/myproject && \
    echo "<?php phpinfo(); ?>" > /var/www/myproject/index.php

# 添加 Nginx 配置文件
COPY test.conf /etc/nginx/http.d/test.conf

# 暴露端口
EXPOSE 80

# 定义工作目录
WORKDIR /var/www/myproject

# 启动 Nginx 和 PHP-FPM
CMD ["sh", "-c", "php-fpm83 && nginx -g 'daemon off;'"]
```

2、创建 Nginx 配置文件

创建一个名为 test.conf 的 Nginx 配置文件。

```bash
server {
    listen 80;
    server_name mydomain.com;  # 替换为你的域名

    root /var/www/myproject;
    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }

    location ~ /\.ht {
        deny all;
    }

    error_log /var/log/nginx/myproject_error.log;
    access_log /var/log/nginx/myproject_access.log;
}
```

3、创建 docker-compose.yml

```yaml
version: '3.8'

services:
  web:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8080:80"
    environment:
      - NGINX_HOST=mydomain.com
    container_name: my-alpine-container
```

4、构建和运行容器

```bash
export DOCKER_BUILDKIT=0
docker-compose up --build
```

5、测试访问

在宿主机的 `/etc/hosts` 中添加条目以支持虚拟域名：

```bash
127.0.0.1 mydomain.com
```

然后在浏览器中通过 http://mydomain.com:8080 访问。

# 练习

1、创建一个名为 xuepingmall-book 的目录

```bash
mkdir xuepingmall-book
```

2、在 xuepingmall-book 目录下新建 Dockerfile 

# 小结

根据上面的两个例子的简单学习，接下来就是在实际使用中进行归纳整理，融汇贯通。
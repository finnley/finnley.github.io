+++
title = 'Docker开发环境搭建'
date = 2022-04-11T10:02:00+08:00
draft = true
categories = [ "Docker" ]
tags = [ "docker" ]
+++

# 环境概述

[github](https://github.com/finnley/env-docker)

`env-docker` 是基于 `docker-compose` 运行在 `Docker` 上的开发环境，包含 `PHP`, `MySQL`, `Nginx`, `Redis` 等镜像，并支持多版本切换，可以满足一些简单的学习，开发和测试需求

**包含镜像**

`docker-env` 包含以下镜像，每种镜像支持多个版本：

- nginx 
- php-fpm (7.4 - 7.3 - 7.2 - 7.1 - 5.6)
- mysql (8.0 - 5.7 - 5.6)
- mongo 
- redis (5.0 - 4.0)
- memcached (1.5.16 - 1.5 - 1)

其中：
php-fpm 默认是 7.1 版本，如需使用其它版本，配置 `.env` 文件中 `PHP_VERSION` 即可；
mysql 默认是 5.7 版本，如需使用其它版本，配置 `.env` 文件中 `MYSQL_VERSION` 即可；

# PHP

## Docker

1.首先创建一个根目录 `env-docker`，进入该目录，在该目录中创建一个 `php-fpm` 的目录

```
mkdir env-docker && cd env-docker
mkdir php-fpm && cd php-fpm
```

2.在 `php-fpm` 目录中新建 `conf-7.1` 目录，进入该目录，创建一系列配置文件,目录结构：

```
.
├── conf-7.1
│   ├── php-fpm.conf
│   ├── php-fpm.d
│   │   ├── docker.conf
│   │   ├── www.conf
│   │   └── zz-docker.conf
│   └── php.ini
```

3.在 `php-fpm` 中新建 `Dockerfile`

[Dockerfile](https://gitee.com/finnley/docker-env/blob/master/php-fpm/Dockerfile)

4.构建 `php-fpm` 镜像

```
docker build --build-arg PHP_VERSION=7.1 -t finnley/php-fpm-7.1 .
```

ARG指令定义参数，在 `docker build` 命令中以 `--build-arg a_name=a_value b_name=b_value` 形式赋值。

```
REPOSITORY                   TAG              IMAGE ID       CREATED         SIZE
finnley/php-fpm-7.1          latest           dc19e45c09dc   2 hours ago     354MB
```

5.创建容器

```
docker run -it -d --name php-fpm \
    -p 9000:9000 \
    -v ${PWD}/conf-7.1/php.ini:/usr/local/etc/php/php.ini \
    -v ${PWD}/conf-7.1/php-fpm.conf:/usr/local/etc/php-fpm.conf \
    -v ${PWD}/conf-7.1/php-fpm.d:/usr/local/etc/php-fpm.d \
    -v ~/Coding:/var/www/site \
    finnley/php-fpm-7.1
```

其中 `~/Coding` 挂载的是我本地的项目目录

6.查看容器运行状态

```
[vagrant@centos7-learning php-fpm]$ docker ps
CONTAINER ID   IMAGE                 COMMAND                  CREATED              STATUS              PORTS                    NAMES
73d6f4f61ad6   finnley/php-fpm-7.1   "docker-php-entrypoi…"   About a minute ago   Up About a minute   0.0.0.0:9000->9000/tcp   php-fpm
[vagrant@centos7-learning php-fpm]$
```

7. 进入容器查看php版本

```
[vagrant@centos7-learning php-fpm]$ docker exec -it 73d6f4f61ad6 /bin/sh
/var/www/site # php -v
PHP 7.1.33 (cli) (built: Oct 25 2019 07:25:49) ( NTS )
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.1.0, Copyright (c) 1998-2018 Zend Technologies
/var/www/site #
```

8. 退出容器 `exit`

## Docker Compose

1.目录结构

```
.
├── README.md
├── docker-compose.yml
├── docs
│   ├── https.md
│   └── vhost.md
├── env-example
├── services
│   ├── php-fpm
│   │   ├── Dockerfile
│   │   ├── conf-7.1
│   │   │   ├── php-fpm.conf
│   │   │   ├── php-fpm.d
│   │   │   │   ├── docker.conf
│   │   │   │   ├── www.conf
│   │   │   │   └── zz-docker.conf
│   │   │   └── php.ini
└── www
    ├── 50x.html
    ├── index.html
    ├── phpinfo.php
```

2.docker-compose.yml

```yml
version: "3"

services:
  #--------------------------------------------------------------------------
  # php-fpm
  #--------------------------------------------------------------------------
  php-fpm:
    container_name: ${ENV}-php-pfm
    build:
      context: ./services/php-fpm
      args:
        - PHP_VERSION=${PHP_VERSION}
        - INSTALL_PCNTL=${PHP_INSTALL_PCNTL}
        - INSTALL_OPCACHE=${PHP_INSTALL_OPCACHE}
        - INSTALL_ZIP=${PHP_INSTALL_ZIP}
        - INSTALL_REDIS=${PHP_INSTALL_REDIS}
        - INSTALL_REDIS_VERSION=${PHP_INSTALL_REDIS_VERSION}
        - INSTALL_MONGODB=${PHP_INSTALL_MONGODB}
        - INSTALL_MONGODB_VERSION=${PHP_INSTALL_MONGODB_VERSION}
        - INSTALL_MEMCACHED=${PHP_INSTALL_MEMCACHED}
        - INSTALL_MEMCACHED_VERSION=${PHP_INSTALL_MEMCACHED_VERSION}
        - INSTALL_SWOOLE=${PHP_INSTALL_SWOOLE}
        - INSTALL_SWOOLE_VERSION=${PHP_INSTALL_SWOOLE_VERSION}
        - INSTALL_YAF=${PHP_INSTALL_YAF}
        - INSTALL_YAF_VERSION=${PHP_INSTALL_YAF_VERSION}
        - INSTALL_XUNSEARCH=${PHP_INSTALL_XUNSEARCH}
        - INSTALL_COMPOSER=${PHP_INSTALL_COMPOSER}
    ports:
      - 9000:9000
    volumes:
      - ./services/php-fpm/conf-${PHP_VERSION}/php.ini:/usr/local/etc/php/php.ini
      - ./services/php-fpm/conf-${PHP_VERSION}/php-fpm.conf:/usr/local/etc/php-fpm.conf
      - ./services/php-fpm/conf-${PHP_VERSION}/php-fpm.d:/usr/local/etc/php-fpm.d
      - ${APP_CODE_PATH_HOST}:/var/www/site
    networks:
      - backend

networks:
  frontend:
    driver: ${NETWORKS_DRIVER}
  backend:
    driver: ${NETWORKS_DRIVER}
  elastic:
    driver: ${NETWORKS_DRIVER}
  rocketmq:
    name: rmq
    driver: ${NETWORKS_DRIVER}
```

`${APP_CODE_PATH_HOST}` 为我项目目录

3.启动

```
docker-compose up
```

4.查看

```
✗ docker ps -a
CONTAINER ID   IMAGE                COMMAND                  CREATED         STATUS         PORTS                    NAMES
634117701f74   env-docker_php-fpm   "docker-php-entrypoi…"   3 minutes ago   Up 3 minutes   0.0.0.0:9000->9000/tcp   dev-php-pfm
✗ docker exec -it 634117701f74 /bin/sh
/var/www/site # php -v
PHP 7.1.33 (cli) (built: Oct 25 2019 07:25:49) ( NTS )
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.1.0, Copyright (c) 1998-2018 Zend Technologies
/var/www/site #
```

# Nginx

## Docker

1.在 `env-docker` 目录中创建一个 `nginx` 的目录，并进入 `nginx` 目录，Docker方式下所有关于 `nginx` 的操作都在当是在 `nginx` 目录中操作

```
mkdir nginx && cd nginx
```

2.`nginx` 目录内容如下

- `conf.d` 目录：用户存在一些 nginx 配置文件，后面会将里面的配置文件挂载到 nginx 的 `/etc/nginx/conf.d` 目录中
- `nginx.conf` 文件：nginx 的主要配置文件，后面会将这个配置文件映射到 nginx 的 `/etc/nginx/nginx.conf` 文件
- `ssl` 目录：配置 `https` 的目录

3.新建 `Dockerfile` 文件

Dockerfile References: [nginx-Dockerfile](https://github.com/laradock/laradock/blob/master/nginx/Dockerfile)

**注意**
如果没有只需要Nginx，不需要PHP，需要将Nginx下的 Dockerfile 文件中的下面代码注释掉，并且还需要将 `sites` 目录下的 `.conf` 结尾的文件修改为非 `.conf` 结尾：
```
# Set upstream conf and remove the default conf
# RUN echo "upstream php-upstream { server ${PHP_UPSTREAM_CONTAINER}:${PHP_UPSTREAM_PORT}; }" > /etc/nginx/conf.d/upstream.conf \
#     && rm /etc/nginx/conf.d/default.conf
```

或者上面内容不注释，修改 `sites/default.conf` 文件，将里面的 `fastcgi_pass php-upstream;` 这一行注释掉

**docker中的Nginx镜像中的nginx:alpine是什么意思?**

相比 `nginx:latest`，`nginx:alpine` 有几点优势：
用的是最新版nginx镜像，功能与 `nginx:latest` 一模一样<`alpine` 镜像用的是 `Alpine Linux` 内核，比ubuntu内核要小很多。另外 `nginx:alpine` 默认支持 `http2`。

4.构建镜像，构建时指定阿里源，下载会快很多

```
docker build --build-arg CHANGE_SOURCE=true -t finnley/nginx .
```

镜像如下：
```
[vagrant@docker-env nginx]$ docker images
REPOSITORY                   TAG       IMAGE ID       CREATED         SIZE
finnley/nginx                latest    17983ab9ccef   6 seconds ago   22.3MB
```

5.创建容器并进入容器

```
docker run -it -d --name web \
    -p 80:80 \
    -v ${PWD}/logs/nginx:/var/log/nginx \
    -v ${PWD}/sites:/etc/nginx/sites-available \
    -v ${PWD}/ssl:/etc/nginx/ssl \
    -v ~/Coding:/var/www/site \
    finnley/nginx
```

如果使用 `PHP`，在上面的基础上添加一句 `--link php-fpm \`:

```
docker run -it -d --name web \
    --link php-fpm \
    -p 80:80 \
    -v ${PWD}/logs/nginx:/var/log/nginx \
    -v ${PWD}/sites:/etc/nginx/sites-available \
    -v ${PWD}/ssl:/etc/nginx/ssl \
    -v ~/Coding:/var/www/site \
    finnley/nginx
```

```
docker ps a
```

如要进入 `alpine` 容器，命令是（后面的路径不是 `/bin/bash`）：

```
$ docker exec -it 55c8d4a47a48 /bin/sh
```

6.在 虚拟主机 `/var/www/site` 目录中新建 `phpinfo.php` 文件

```
<?php
    echo "Hello, PHP!!!";
    phpinfo();
```

7.浏览器中输入 `http://127.0.0.1/phpinfo.php`

## Docker Compose

**不关联PHP**
```yaml
nginx:
    container_name: ${ENV}-nginx
    build:
      context: ./services/nginx
      args:
        - CHANGE_SOURCE=${CHANGE_SOURCE}
        - PHP_UPSTREAM_CONTAINER=${NGINX_PHP_UPSTREAM_CONTAINER}
        - PHP_UPSTREAM_PORT=${NGINX_PHP_UPSTREAM_PORT}
        - http_proxy
        - https_proxy
        - no_proxy
    volumes:
      - ./services/nginx/logs/nginx/:/var/log/nginx
      - ./services/nginx/sites/:/etc/nginx/sites-available
      - ./services/nginx/ssl/:/etc/nginx/ssl
      # - ./services/nginx/nginx.conf:/etc/nginx/nginx.conf
      - ${APP_CODE_PATH_HOST}:/var/www/site
    ports:
      - 80:80
      - 81:81
      - 443:443
    # environment:
    #   - BACKEND_API_DOMAIN=${BACKEND_API_DOMAIN}
    working_dir: /var/www/site
    tty: true
    networks:
      - frontend
      - backend
```

**关联PHP**
```yaml
nginx:
    container_name: ${ENV}-nginx
    build:
      context: ./services/nginx
      args:
        - CHANGE_SOURCE=${CHANGE_SOURCE}
        - PHP_UPSTREAM_CONTAINER=${NGINX_PHP_UPSTREAM_CONTAINER}
        - PHP_UPSTREAM_PORT=${NGINX_PHP_UPSTREAM_PORT}
        - http_proxy
        - https_proxy
        - no_proxy
    volumes:
      - ./services/nginx/logs/nginx/:/var/log/nginx
      - ./services/nginx/sites/:/etc/nginx/sites-available
      - ./services/nginx/ssl/:/etc/nginx/ssl
      # - ./services/nginx/nginx.conf:/etc/nginx/nginx.conf
      - ${APP_CODE_PATH_HOST}:/var/www/site
    ports:
      - 80:80
      - 81:81
      - 443:443
    # environment:
    #   - BACKEND_API_DOMAIN=${BACKEND_API_DOMAIN}
    working_dir: /var/www/site
    tty: true
    depends_on:
      - php-fpm
    networks:
      - frontend
      - backend
```


# MySQL

## 原生Docker

1. 在 `docker-env` 目录中创建一个 `mysql` 的目录
2. 新建 `Dockerfile` 文件

```
ARG MYSQL_VERSION
FROM mysql:${MYSQL_VERSION}

LABEL maintainer="Finnley <43@43.com>"

CMD ["mysqld"]

EXPOSE 3306
```

3. 构建 `MySQL` 镜像

```
docker build --build-arg MYSQL_VERSION=5.7 -t finnley/mysql .
```

4. 创建 `MySQL` 容器

```
docker run -d -p 3306:3306 --name mysql -v $PWD/conf:/etc/mysql/conf.d -v $PWD/logs:/logs -v ${PWD}/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123 finnley/mysql
```

5. 查看容器状态

```
[vagrant@centos7-learning mysql]$ docker ps
CONTAINER ID   IMAGE                 COMMAND                  CREATED         STATUS         PORTS                               NAMES
f416cd43ec30   finnley/mysql         "docker-entrypoint.s…"   6 seconds ago   Up 6 seconds   0.0.0.0:3306->3306/tcp, 33060/tcp   mysql
73d6f4f61ad6   finnley/php-fpm-7.1   "docker-php-entrypoi…"   6 minutes ago   Up 6 minutes   0.0.0.0:9000->9000/tcp              php-fpm
[vagrant@centos7-learning mysql]$
```

6. 进入容器

```
[vagrant@centos7-learning mysql]$ docker exec -it f416cd43ec30 /bin/sh
# mysql -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.33 MySQL Community Server (GPL)

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.00 sec)

mysql>
```

7. 远程连接测试

经使用 `Navicat` DBMS 管理软件进行连接测试输入对应 vagrant IP 和 MySQL 密码成功连接，截图略



## Mongo

1. 在 `docker-env` 目录中创建一个 `mongo` 的目录，里面新建 `mongo-conf` 目录，并在 `mongo-conf` 里面新建一个 `yapi` 的初始化数据文件 `init-yapi-mongo.js`

内容如下:

```
db.createUser({ user: 'admin', pwd: 'admin', roles: [ { role: "root", db: "admin" } ] });

db.auth("admin", "admin");
db.createUser({
    user: 'yapi',
    pwd: '123',
    roles: [
        { role: "dbAdmin", db: "yapi" },
        { role: "readWrite", db: "yapi" }
    ]
});
```

2. 在 `mongo` 目录下新建 `Dockerfile` 文件

```
ARG MONGO_VERSION
FROM mongo:${MONGO_VERSION}

LABEL maintainer="Finnley <43@43.com>"

#COPY mongo.conf /usr/local/etc/mongo/mongo.conf

CMD ["mongod"]

EXPOSE 27017
```

3. `docker-compose.yml` 文件添加 `mongo` 容器

```
...
  mongo:
    container_name: dev-mongo
    build:
      context: ./mongo
      args:
        - MONGO_VERSION=${MONGO_VERSION}
    ports:
      - ${MONGO_PORT}:27017
    volumes:
      - ${PWD}/mongo/mongo-conf:/docker-entrypoint-initdb.d
      - ${PWD}/mongo/etc:/etc/mongo
      - ${PWD}/mongo/data/db:/data/db
    restart: always
    environment:
      MONGO_INITDB_ROOT_USERNAME: ${MONGO_ROOT_USERNAME}
      MONGO_INITDB_ROOT_PASSWORD: ${MONGO_ROOT_PASSWORD}
      MONGO_INITDB_DATABASE: ${MONGO_INITDB_DATABASE}
    healthcheck:
      test: ["CMD", "netstat -anp | grep 27017"]
      interval: 2m
      timeout: 10s
      retries: 3
```

## YApi

在根目录 `docker-env` 目录下新建 `yapi` 的目录，在下面新建 `Dockerfile` 和 `repositories` 两个文件

1. Dockerfile

```
FROM node:12-alpine

COPY repositories /etc/apk/repositories

# RUN npm install -g yapi-cli --registry http://mirrors.cloud.tencent.com/npm/
RUN npm install -g yapi-cli --registry https://registry.npm.taobao.org

EXPOSE 3000 9090
```

2. repositories

```
https://mirrors.aliyun.com/alpine/v3.6/main/

https://mirrors.aliyun.com/alpine/v3.6/community/
```

3. `docker-compose.yml` 文件添加 `yapi` 容器

```
...
  yapi:
    container_name: dev-yapi
    build:
      context: ./yapi
      dockerfile: Dockerfile
    image: yapi
    # 第一次启动使用
    command: "yapi server"
    # 之后使用下面的命令
    # command: "node /my-yapi/vendors/server/app.js"
    volumes: 
      - ${PWD}/yapi/my-yapi:/my-yapi
    ports: 
      - 9090:9090
      - 3000:3000
    depends_on: 
      - mongo
```

4. yapi 部署

当使用 `docker-compose up` 的加载的时候屏幕中看到 

```
dev-yapi   |  初始化管理员账号成功,账号名："admin@admin.com"，密码："ymfe.org"
dev-yapi   |
dev-yapi   | 部署成功，请切换到部署目录，输入： "node vendors/server/app.js" 指令启动服务器, 然后在浏览器打开 http://127.0.0.1:3000 访问
```

打开 `localhost:9090`

* 默认部署路径为 `/my-yapi` (需要修改 `docker-compose.yml` 才可以更改)
* 修改管理员邮箱 `admin@admin.com` (随意, 修改为自己的邮箱)
* 修改数据库地址为 `mongo` 或者修改为自己的 `mongo` 实例 (`docker-compose` 配置的 `mongo` 服务名称叫 `mongo`)
* 打开数据库认证
* 输入数据库用户名: `yapi` (`mongo` 配置的用户名, 见 `mongo-conf/init-yapi-mongo.js`)
* 输入密码: `123` (`mongo` 配置的密码, 见 `mongo-conf/init-yapi-mongo.js`)

点击开始部署.

* 初始化页面

![](https://images.notes.xuepincat.com/yapi/1.jpg)

* 信息设置

![](https://images.notes.xuepincat.com/yapi/2.png)

* 部署中

![](https://images.notes.xuepincat.com/yapi/3.jpg)

* 部署完成

![](https://images.notes.xuepincat.com/yapi/4.png)

* 修改 `docker-compose.yml` 文件 

```
...
  yapi:
    container_name: dev-yapi
    build:
      context: ./yapi
      dockerfile: Dockerfile
    image: yapi
    # 第一次启动使用
    # command: "yapi server"
    # 之后使用下面的命令
    command: "node /my-yapi/vendors/server/app.js"
    volumes: 
      - ${PWD}/yapi/my-yapi:/my-yapi
    ports: 
      - 9090:9090
      - 3000:3000
    depends_on: 
      - mongo
```

* 退出 `api` 重新启动

```
docker-compose up
```

* 访问 `yapi`，此时访问的端口就不是 `9090`，而是 `3000`， 即 `IP:3000`

![](https://images.notes.xuepincat.com/yapi/5.png)

* 登录

username: admin@admin.com
password: ymfe.org


## 使用 `docker-compose` 构建容器

1. 在 `docker-lnmp` 目录中新建 `.env` 文件,用户设置一些环境变量信息

```
vim .env
```

内容如下：

```
### Paths #################################################

WEB_ROOT_PATH=../
```

2. 在 `docker-lnmp` 目录中新建 `docker-compose.yml` 文件

```
vim docker-compose.yml
```

3. 后台启动

```
docker-compose up -d
```

或者 

```
docker-compose up -d nginx
```

[docker-compose.yml](https://github.com/finnley/docker-dev/blob/dev/docker-compose.yml)

# Git安装与配置

[Git安装与配置](https://notes.einscat.com/2017/03/27/Git%E5%AE%89%E8%A3%85%E4%B8%8E%E9%85%8D%E7%BD%AE/)

**概述**：
1.Git版本管理工具的图形界面安装与编译安装

## 使用 docker 安装 MySQL

## 下载镜像 

```
docker pull mysql:5.7
```

可以通过 `docker images` 命令来查看拉取的镜像

```
[vagrant@dev-centos7 ~]$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED      SIZE
mysql        5.7       697daaecf703   8 days ago   448MB
[vagrant@dev-centos7 ~]$
```

## 通过镜像启动容器

```
docker run -p 3306:3306 --name mysql -v $PWD/conf:/etc/mysql/conf.d -v $PWD/logs:/logs -v $PWD/mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123 -d mysql:5.7
```

* `-p 3306:3306`：将容器的 3306 端口映射到主机的 3306 端口。
* `-v $PWD/conf:/etc/mysql/conf.d`：将主机当前目录下的 `conf/my.cnf` 挂载到容器的 `/etc/mysql/my.cnf`。
* `-v $PWD/logs:/logs`：将主机当前目录下的 `logs` 目录挂载到容器的 `/logs`。
* `-v $PWD/mysql-data:/var/lib/mysql` ：将主机当前目录下的 `mysql-data` 目录挂载到容器的 `/var/lib/mysql` 。
* `-e MYSQL_ROOT_PASSWORD=123`：初始化 `root` 用户的密码。

通过 `docker ps -a` 查看启动的容器

```
[vagrant@dev-centos7 ~]$ docker ps -a
CONTAINER ID   IMAGE       COMMAND                  CREATED          STATUS          PORTS                               NAMES
de09af0e79cd   mysql:5.7   "docker-entrypoint.s…"   48 seconds ago   Up 46 seconds   0.0.0.0:3306->3306/tcp, 33060/tcp   mysql
[vagrant@dev-centos7 ~]$
```

## 配置

由于 `MySQL` 的安全策略，现在还不能使用 `root/123`来访问数据库

1. 进入容器

通过 `docker ps -a` 查看 `mysql` 的容器 `id` 然后使用：

格式：`docker exec -it {Container ID} /bin/bash`

比如：

```
docker exec -it de09af0e79cd /bin/bash
```

2. 登录 MySQL ，进入 `MySQL` 交互环境


```
mysql -uroot -p123
```

3. 建立用户授权

```
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'root' WITH GRANT OPTION;
```

第一句表示让所有的IP地址都可以访问所有的表，用户为 `root`,密码也是 `root`,数据库之前启动的密码是 `123`，现在新建了一个用户名是 `root`,密码也是 `root` 的用户s

```
GRANT ALL PRIVILEGES ON *.* TO 'root'@'127.0.0.1' IDENTIFIED BY 'root' WITH GRANT OPTION;
```

```
GRANT ALL PRIVILEGES ON *.* TO 'root'@'localhost' IDENTIFIED BY 'root' WITH GRANT OPTION;
```

```
FLUSH PRIVILEGES;
```

4. `exit` 可以退出容器，第一次输入 `exit` 是退出 `MySQL`,第二次输入 `exit` 是退出容器，然后使用 `Navicat` 连接 MySQL
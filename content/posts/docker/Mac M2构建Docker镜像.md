+++
title = 'Mac M2构建Docker镜像'
date = 2024-10-23T22:27:15+08:00
draft = true
+++

# Buildx 是什么

- 使用 Docker Buildx 可以构建多种系统架构下的 Docker 镜像。
- 它是一个 Docker CLI 插件。
- Docker for Mac 默认自带。
- 支持 Docker v19.03+ 以上版本。

# 使用

1、环境

Macbook M2

2、检查是否启用 Buildx

```bash
➜  ~ docker buildx version
github.com/docker/buildx v0.11.2-desktop.1 986ab6afe790e25f022969a18bc0111cff170bc2
```

3、创建一个新的 Buildx builder

默认情况下是没有启用buildx配置的, 需要自己创建一个，创建一个名为 “mybuilder” 的 Buildx builder，并将其设置为当前使用的 builder:

```bash
docker buildx create --name mybuilder --use
docker buildx inspect --bootstrap
```

4、使用新建立的 builder 建立 Docker Image。在 Dockerfile 所在的目录中行以下命令：

```bash
docker buildx build --platform linux/amd64 -t Image-name:tag .
```

`–platform linux/amd64` 指定要建立的架构为 x86 。


apk update
apk add bash curl nano vim

apk add nginx php php-fpm php-cli

cat > /etc/nginx/conf.d/test.conf <<'EOF'
server {
    listen 80;
    server_name mydomain.com;  # 替换为你的域名

    root /var/www/html;
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

echo "<?php phpinfo(); ?>" > /var/www/html/index.php
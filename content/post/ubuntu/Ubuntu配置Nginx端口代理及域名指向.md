+++
title = 'Ubuntu配合Vultr搭建ShadowsocksUbuntu配置Nginx端口代理及域名指向'
date = 2018-09-04T00:36:19+08:00
draft = true
categories = [ "Ubuntu" ]
tags = [ "ubuntu" ]
+++

## 安装 `Nginx`

```
sudo apt-get install nginx
```

## 查看 `Nginx` 版本

当前版本 1.10.3

```
nginx -v
```

<div align=center>
![](/images/node/20180904/1.png)
</div>

进入 `nginx` 目录

```
cd /etc/nginx/conf.d/
```

```
sudo vi ice-ornnth-xyz-3000.conf
```

```
upstream ice {
        server 127.0.0.1:3000;
}

server {
        listen 80;
        server_name ice.ornnth.xyz;

        location / {
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forward-For $proxy_add_x_forwarded_for;
                proxy_set_header Host $http_host;
                proxy_set_header X-Nginx-Proxy true;

                proxy_pass http://ice;
                proxy_redirect off;
        }

        #location ~* ^.+\.(jpg|jpeg|gif|png|ico|css|js|pdf|txt) {
                #root /www/ice/production/current/public;
        #}
}

```

然后去添加一条域名解析记录

<div align=center>
![](/images/node/20180904/2.png)
</div>

<div align=center>
![](/images/node/20180904/3.png)
</div>

## 重启 `Nginx`

```
sudo service nginx restart
```

<div align=center>
![](/images/node/20180904/4.png)
</div>

## 测试

打开浏览器，在地址栏中输入域名 `ice.ornnth.xyz`后回车

<div align=center>
![](/images/node/20180904/5.png)
</div>

## 关闭 `nginx` 版本
到这里就可以通过域名来访问NodeJS服务了，但是打开调试工具会看到 `nginx` 的版本

<div align=center>
![](/images/node/20180904/6.png)
</div>

```
sudo vi /etc/nginx/nginx.conf
```

找到下面的一句话，把前面的 `#` 去掉，将 `server_tokens` 设置为 `off` ,这样就不会将`Nginx`版本号输出到前端了

```
# server_tokens off;
```

重启 `Nginx`

```
sudo service nginx restart
```


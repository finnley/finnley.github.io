+++
title = '基于Docker搭建harbor私有镜像仓库'
date = 2024-06-27T18:32:02+08:00
draft = false
categories = [ "Linux" ]
+++

## 下载

```bash
cd /opt
wget https://github.com/goharbor/harbor/releases/download/v2.11.0/harbor-offline-installer-v2.11.0.tgz
```

## 解压

```bash
# tar zxvf harbor-offline-installer-v2.11.0.tgz
harbor/harbor.v2.11.0.tar.gz
harbor/prepare
harbor/LICENSE
harbor/install.sh
harbor/common.sh
harbor/harbor.yml.tmpl
```

## 配置

```bash
cd /opt/harbor
# 复制一份harbor的配置文件并改名harbor.yml
cp -ar harbor.yml.tmpl harbor.yml

# vim 进入配置文件
vim harbor.yml
```

修改 `hostname`，这里配置为监听地址，也可以是域名：
```yaml
...
# The IP address or hostname to access admin UI and registry service.
# DO NOT use localhost or 127.0.0.1, because Harbor needs to be accessed by external clients.
hostname: 10.186.62.66
...
```

修改 `port`:
```yaml
...
# http related config
http:
  # port for http, default is 80. If https enabled, this port will redirect to https port
  port: 10010
...
```
 

修改 `harbor_admin_password` ， 配置 admin 用户的密码：
```yaml
# The initial password of Harbor admin
# It only works in first time to install harbor
# Remember Change the admin password from UI after launching Harbor.
harbor_admin_password: Harbor12345
```

注意 `It only works in first time to install harbor` 这句话，是说设置的密码仅在第一次安装 Harbor 时生效，之后再重新销毁容器，重新设置密码，重新部署都将不会生效。但是可以进入 Harbor 在页面中修改 admin 用户的密码。
 

修改 `data_volume`，设置数据仓库：
```yaml
...
# The default data volume
data_volume: /data/harbor
...
```
 
注释 `https`，在13行开始：
```bash
...
# https related config
#https:
  # https port for harbor, default is 443
  #port: 443
  # The path of cert and key files for nginx
  #certificate: /your/certificate/path
  #private_key: /your/private/key/path
...
```

## 安装

```
# Harbor安装环境预处理
./prepare
# 安装并启动Harbor
./install.sh
# 检查是否安装成功（应该是启动9个容器）要在harbor目录下操作，否则docker-compose会又问题；
docker-compose ps

```

过程如下：
```bash
# ./prepare
prepare base dir is set to /opt/harbor
Unable to find image 'goharbor/prepare:v2.11.0' locally
v2.11.0: Pulling from goharbor/prepare
2a715ab8d96e: Pull complete
a4464fa7f83c: Pull complete
e614c191ff97: Pull complete
c7c6a92d495e: Pull complete
8a1870f4fc3a: Pull complete
476340c45540: Pull complete
daddd88513e3: Pull complete
76f33058ccdc: Pull complete
9cc7b9f5c87d: Pull complete
58fedf6bbb75: Pull complete
Digest: sha256:b7062a3af02c1f32100e0da3576433b5cd419793039779cb3353c4af0d4f9a62
Status: Downloaded newer image for goharbor/prepare:v2.11.0
WARNING:root:WARNING: HTTP protocol is insecure. Harbor will deprecate http protocol in the future. Please make sure to upgrade to https
Generated configuration file: /config/portal/nginx.conf
Generated configuration file: /config/log/logrotate.conf
Generated configuration file: /config/log/rsyslog_docker.conf
Generated configuration file: /config/nginx/nginx.conf
Generated configuration file: /config/core/env
Generated configuration file: /config/core/app.conf
Generated configuration file: /config/registry/config.yml
Generated configuration file: /config/registryctl/env
Generated configuration file: /config/registryctl/config.yml
Generated configuration file: /config/db/env
Generated configuration file: /config/jobservice/env
Generated configuration file: /config/jobservice/config.yml
Generated and saved secret to file: /data/secret/keys/secretkey
Successfully called func: create_root_cert
Generated configuration file: /compose_location/docker-compose.yml
Clean up the input dir
#

# docker images
REPOSITORY           TAG           IMAGE ID       CREATED        SIZE
goharbor/prepare     v2.11.0       2baf15fbf5e2   3 weeks ago    207MB
```

```bash
# ./install.sh

[Step 0]: checking if docker is installed ...

Note: docker version: 26.1.3

[Step 1]: checking docker-compose is installed ...

Note: Docker Compose version v2.27.0

[Step 2]: loading Harbor images ...
c1bfdd037444: Loading layer [==================================================>]  11.62MB/11.62MB
f875e254b9ac: Loading layer [==================================================>]  3.584kB/3.584kB
f6396eddeec1: Loading layer [==================================================>]   2.56kB/2.56kB
1ebf5d87fa7d: Loading layer [==================================================>]  67.07MB/67.07MB
fc1e5bb45640: Loading layer [==================================================>]  5.632kB/5.632kB
e44697797ac0: Loading layer [==================================================>]  125.4kB/125.4kB
c0ae68cf7be1: Loading layer [==================================================>]  201.7kB/201.7kB
726f07deec0d: Loading layer [==================================================>]  68.19MB/68.19MB
3a686a2eff08: Loading layer [==================================================>]   2.56kB/2.56kB
Loaded image: goharbor/harbor-core:v2.11.0
ac184afac0b6: Loading layer [==================================================>]  16.19MB/16.19MB
8611a95c6e66: Loading layer [==================================================>]  175.1MB/175.1MB
448930a6ff31: Loading layer [==================================================>]  26.02MB/26.02MB
290df6c1ce06: Loading layer [==================================================>]  18.43MB/18.43MB
578736bb7ff2: Loading layer [==================================================>]   5.12kB/5.12kB
8435240e3c9b: Loading layer [==================================================>]  6.144kB/6.144kB
b68e043688c9: Loading layer [==================================================>]  3.072kB/3.072kB
dabdc5c3cac4: Loading layer [==================================================>]  2.048kB/2.048kB
702b704106ad: Loading layer [==================================================>]   2.56kB/2.56kB
b5638d9a9b77: Loading layer [==================================================>]   7.68kB/7.68kB
Loaded image: goharbor/harbor-db:v2.11.0
9ca57b2b588b: Loading layer [==================================================>]  115.6MB/115.6MB
Loaded image: goharbor/nginx-photon:v2.11.0
ab78d66932a9: Loading layer [==================================================>]  9.117MB/9.117MB
1956e92963ef: Loading layer [==================================================>]  4.096kB/4.096kB
10b07c4be20e: Loading layer [==================================================>]  3.072kB/3.072kB
54dad0c9719c: Loading layer [==================================================>]  219.8MB/219.8MB
cf7626b04a7c: Loading layer [==================================================>]  14.54MB/14.54MB
778b7bf507d5: Loading layer [==================================================>]  235.1MB/235.1MB
Loaded image: goharbor/trivy-adapter-photon:v2.11.0
965b313a9c10: Loading layer [==================================================>]  16.19MB/16.19MB
dc70a1b84845: Loading layer [==================================================>]  110.6MB/110.6MB
c95724e107ee: Loading layer [==================================================>]  3.072kB/3.072kB
7d8561b7288f: Loading layer [==================================================>]   59.9kB/59.9kB
ff506c6907f9: Loading layer [==================================================>]  61.95kB/61.95kB
Loaded image: goharbor/redis-photon:v2.11.0
73f4adf776bc: Loading layer [==================================================>]  8.607MB/8.607MB
c0a401068ccf: Loading layer [==================================================>]  4.096kB/4.096kB
a57486c4c947: Loading layer [==================================================>]  3.072kB/3.072kB
8b62b878cb79: Loading layer [==================================================>]  17.86MB/17.86MB
98d7c2db6a33: Loading layer [==================================================>]  18.65MB/18.65MB
Loaded image: goharbor/registry-photon:v2.11.0
Loaded image: goharbor/prepare:v2.11.0
0590427bac2a: Loading layer [==================================================>]  115.6MB/115.6MB
f9daca73c9e1: Loading layer [==================================================>]  6.701MB/6.701MB
5a6046ceb202: Loading layer [==================================================>]  249.3kB/249.3kB
bbd38db3ed9f: Loading layer [==================================================>]  1.477MB/1.477MB
Loaded image: goharbor/harbor-portal:v2.11.0
42a378dba88b: Loading layer [==================================================>]  125.3MB/125.3MB
86358bf40806: Loading layer [==================================================>]  3.584kB/3.584kB
6040ece11cad: Loading layer [==================================================>]  3.072kB/3.072kB
24e21bd8a0e3: Loading layer [==================================================>]   2.56kB/2.56kB
2b63c3625035: Loading layer [==================================================>]  3.072kB/3.072kB
fee5734cbad0: Loading layer [==================================================>]  3.584kB/3.584kB
185875a0e998: Loading layer [==================================================>]  20.48kB/20.48kB
Loaded image: goharbor/harbor-log:v2.11.0
3e3728dfc1a7: Loading layer [==================================================>]  11.62MB/11.62MB
154976c83098: Loading layer [==================================================>]  3.584kB/3.584kB
15c1c2349bcb: Loading layer [==================================================>]   2.56kB/2.56kB
85dc9064eb8d: Loading layer [==================================================>]  54.27MB/54.27MB
97105abb6f9b: Loading layer [==================================================>]  55.06MB/55.06MB
Loaded image: goharbor/harbor-jobservice:v2.11.0
a3d6a21f644b: Loading layer [==================================================>]  8.607MB/8.607MB
6db23eb33a69: Loading layer [==================================================>]  4.096kB/4.096kB
3126cf040566: Loading layer [==================================================>]  17.86MB/17.86MB
4f0a8f24f823: Loading layer [==================================================>]  3.072kB/3.072kB
b020cb5c3c24: Loading layer [==================================================>]  38.92MB/38.92MB
bcd03615fa0e: Loading layer [==================================================>]  57.57MB/57.57MB
Loaded image: goharbor/harbor-registryctl:v2.11.0
735f3382b7bb: Loading layer [==================================================>]  11.62MB/11.62MB
8f451ba6451b: Loading layer [==================================================>]  28.74MB/28.74MB
cef73f8e1547: Loading layer [==================================================>]  4.608kB/4.608kB
905598d5c2a3: Loading layer [==================================================>]  29.53MB/29.53MB
Loaded image: goharbor/harbor-exporter:v2.11.0


[Step 3]: preparing environment ...

[Step 4]: preparing harbor configs ...
prepare base dir is set to /opt/harbor
WARNING:root:WARNING: HTTP protocol is insecure. Harbor will deprecate http protocol in the future. Please make sure to upgrade to https
Clearing the configuration file: /config/log/logrotate.conf
Clearing the configuration file: /config/log/rsyslog_docker.conf
Clearing the configuration file: /config/db/env
Clearing the configuration file: /config/registry/passwd
Clearing the configuration file: /config/registry/config.yml
Clearing the configuration file: /config/jobservice/config.yml
Clearing the configuration file: /config/jobservice/env
Clearing the configuration file: /config/core/app.conf
Clearing the configuration file: /config/core/env
Clearing the configuration file: /config/portal/nginx.conf
Clearing the configuration file: /config/registryctl/config.yml
Clearing the configuration file: /config/registryctl/env
Clearing the configuration file: /config/nginx/nginx.conf
Generated configuration file: /config/portal/nginx.conf
Generated configuration file: /config/log/logrotate.conf
Generated configuration file: /config/log/rsyslog_docker.conf
Generated configuration file: /config/nginx/nginx.conf
Generated configuration file: /config/core/env
Generated configuration file: /config/core/app.conf
Generated configuration file: /config/registry/config.yml
Generated configuration file: /config/registryctl/env
Generated configuration file: /config/registryctl/config.yml
Generated configuration file: /config/db/env
Generated configuration file: /config/jobservice/env
Generated configuration file: /config/jobservice/config.yml
loaded secret from file: /data/secret/keys/secretkey
Generated configuration file: /compose_location/docker-compose.yml
Clean up the input dir


Note: stopping existing Harbor instance ...
WARN[0000] /opt/harbor/docker-compose.yml: `version` is obsolete


[Step 5]: starting Harbor ...
WARN[0000] /opt/harbor/docker-compose.yml: `version` is obsolete
[+] Running 10/10
 ✔ Network harbor_harbor        Created                                                                                                                                                                                            0.1s
 ✔ Container harbor-log         Started                                                                                                                                                                                            0.6s
 ✔ Container registryctl        Started                                                                                                                                                                                            1.2s
 ✔ Container harbor-db          Started                                                                                                                                                                                            1.0s
 ✔ Container registry           Started                                                                                                                                                                                            1.0s
 ✔ Container redis              Started                                                                                                                                                                                            0.9s
 ✔ Container harbor-portal      Started                                                                                                                                                                                            1.2s
 ✔ Container harbor-core        Started                                                                                                                                                                                            1.3s
 ✔ Container harbor-jobservice  Started                                                                                                                                                                                            1.7s
 ✔ Container nginx              Started                                                                                                                                                                                            1.7s
✔ ----Harbor has been installed and started successfully.----
[root@VM-0-2-centos harbor]#
```

```bash
# docker-compose ps
WARN[0000] /opt/harbor/docker-compose.yml: `version` is obsolete
NAME                IMAGE                                 COMMAND                  SERVICE       CREATED          STATUS                             PORTS
harbor-core         goharbor/harbor-core:v2.11.0          "/harbor/entrypoint.…"   core          32 seconds ago   Up 30 seconds (healthy)
harbor-db           goharbor/harbor-db:v2.11.0            "/docker-entrypoint.…"   postgresql    32 seconds ago   Up 31 seconds (healthy)
harbor-jobservice   goharbor/harbor-jobservice:v2.11.0    "/harbor/entrypoint.…"   jobservice    32 seconds ago   Up 25 seconds (health: starting)
harbor-log          goharbor/harbor-log:v2.11.0           "/bin/sh -c /usr/loc…"   log           32 seconds ago   Up 31 seconds (healthy)            127.0.0.1:1514->10514/tcp
harbor-portal       goharbor/harbor-portal:v2.11.0        "nginx -g 'daemon of…"   portal        32 seconds ago   Up 31 seconds (healthy)
nginx               goharbor/nginx-photon:v2.11.0         "nginx -g 'daemon of…"   proxy         32 seconds ago   Up 30 seconds (healthy)            0.0.0.0:10010->8080/tcp, :::10010->8080/tcp
redis               goharbor/redis-photon:v2.11.0         "redis-server /etc/r…"   redis         32 seconds ago   Up 31 seconds (healthy)
registry            goharbor/registry-photon:v2.11.0      "/home/harbor/entryp…"   registry      32 seconds ago   Up 31 seconds (healthy)
registryctl         goharbor/harbor-registryctl:v2.11.0   "/home/harbor/start.…"   registryctl   32 seconds ago   Up 31 seconds (healthy)
#
```

## 修改 docker 配置

```bash
# 由于docker默认不允许使用非https方式推送镜像，所以在需要pull镜像的服务器配置访问地址
vim /etc/docker/daemon.json
```

内容如下：
```json
{
    ...
    "insecure-registries": [
        "10.186.62.66:10010"
    ]
}
```

注意：这里设置的docker客户端，如自己的电脑，因为不设置在自己电脑上推送镜像会提示如下错误：
```bash
docker push 10.186.62.66:10010/library/busybox:v1
The push refers to repository [10.186.60.66:10010/library/busybox]
Get "https://10.186.60.66:10010/v2/": http: server gave HTTP response to HTTPS client
``

然后重启docker和harbor容器，要在harbor目录下操作，否则docker-compose会有问题:
```bash
systemctl daemon-reload && service docker restart
docker-compose restart
```

docker 登录:
```bash
docker login 10.186.62.66:10010
#或者 
docker login -uadmin -pHaarbor12345 10.186.62.66:10010
```

Hello@harbor#4343

## 页面访问

访问地址：[配置文件中配置的hostname和port] <br>
用户名：admin <br>	
密码：[配置文件配置的密码]

![](/img/linux/harbor/10.jpg)

## 推送镜像

1、测试镜像
```bash
# docker images
REPOSITORY                                               TAG                                        IMAGE ID       CREATED         SIZE
busybox                                                  latest                                     71a676dd070f   2 years ago     1.41MB
```

2、为待上传的镜像打标签(修改镜像名)

注意：**一定要加上项目名称**

格式：
```bash
docker tag 镜像名:版本 your-ip:端口/项目名称/新的镜像名:版本
```

例如：
```bash
# docker tag busybox:latest 111.231.87.78:10010/library/busybox:v1
# docker images
REPOSITORY                                               TAG                                        IMAGE ID       CREATED         SIZE
111.231.87.78:10010/library/busybox                      v1                                         71a676dd070f   2 years ago     1.41MB
busybox                                                  latest                                     71a676dd070f   2 years ago     1.41MB
```

3、推送镜像到 harbor 仓库
格式：
```bash
docker push 修改的镜像名
```

例如：
```bash
#  docker push 111.231.87.78:10010/library/busybox:v1
The push refers to repository [111.231.87.78:10010/library/busybox]
468ad4d964cd: Pushed
v1: digest: sha256:a77fe109c026308f149d36484d795b42efe0fd29b332be9071f63e1634c36ac9 size: 527
```

4、查看

进入 web 页面查看镜像推送结果：
![](/img/linux/harbor/20.jpg)

`library` 是默认的项目名称，点击进去查看刚刚推送的镜像：
![](/img/linux/harbor/30.jpg)

## 拉取镜像

拉取方法一

点击点击这个小按钮，然后直接粘贴，执行拉取命令：
![](/img/linux/harbor/40.jpg)

```bash
docker pull 10.186.62.66:10010/library/busybox@sha256:a77fe109c026308f149d36484d795b42efe0fd29b332be9071f63e1634c36ac9
```

拉取方法二
格式：
```bash
docker pull 上传时修改的镜像名
```

例如：
```bash
# docker pull 10.186.62.66:10010/library/busybox:v1
v1: Pulling from library/busybox
a01966dde7f8: Pull complete
Digest: sha256:a77fe109c026308f149d36484d795b42efe0fd29b332be9071f63e1634c36ac9
Status: Downloaded newer image for 111.231.87.78:10010/library/busybox:v1
10.186.62.66:10010/library/busybox:v1
# docker images
REPOSITORY                            TAG       IMAGE ID       CREATED       SIZE
10.186.62.66:10010/library/busybox   v1        71a676dd070f   2 years ago   1.41MB
```

## 删除镜像

![](/img/linux/harbor/50.jpg)
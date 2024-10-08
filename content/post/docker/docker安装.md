+++
title = 'Docker安装'
date = 2024-04-12T22:56:44+08:00
draft = true
categories = [ "Docker" ]
tags = [ "docker" ]
+++

Docker CE是免费的Docker产品的新名称，Docker CE包含了完整的Docker平台，非常适合开发人员和运维团队构建容器APP。

## CentOS 7（使用 yum 进行安装）

```
# step 1: 安装必要的一些系统工具
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
# Step 2: 添加软件源信息
sudo yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# Step 3
sudo sed -i 's+download.docker.com+mirrors.aliyun.com/docker-ce+' /etc/yum.repos.d/docker-ce.repo
# Step 4: 更新并安装Docker-CE
sudo yum makecache fast
sudo yum -y install docker-ce
# Step 5: 开启Docker服务
sudo service docker start
# Step 6: 开机自启动
sudo systemctl enable docker.service

# 注意：
# 官方软件源默认启用了最新的软件，您可以通过编辑软件源的方式获取各个版本的软件包。例如官方并没有将测试版本的软件源置为可用，您可以通过以下方式开启。同理可以开启各种测试版本等。
# vim /etc/yum.repos.d/docker-ce.repo
#   将[docker-ce-test]下方的enabled=0修改为enabled=1
#
# 安装指定版本的Docker-CE:
# Step 1: 查找Docker-CE的版本:
# yum list docker-ce.x86_64 --showduplicates | sort -r
#   Loading mirror speeds from cached hostfile
#   Loaded plugins: branch, fastestmirror, langpacks
#   docker-ce.x86_64            17.03.1.ce-1.el7.centos            docker-ce-stable
#   docker-ce.x86_64            17.03.1.ce-1.el7.centos            @docker-ce-stable
#   docker-ce.x86_64            17.03.0.ce-1.el7.centos            docker-ce-stable
#   Available Packages
# Step2: 安装指定版本的Docker-CE: (VERSION例如上面的17.03.0.ce.1-1.el7.centos)
# sudo yum -y install docker-ce-[VERSION]

```


## 安装校验

```
# docker version
Client: Docker Engine - Community
 Version:           26.0.1
 API version:       1.45
 Go version:        go1.21.9
 Git commit:        d260a54
 Built:             Thu Apr 11 10:56:30 2024
 OS/Arch:           linux/amd64
 Context:           default

Server: Docker Engine - Community
 Engine:
  Version:          26.0.1
  API version:      1.45 (minimum version 1.24)
  Go version:       go1.21.9
  Git commit:       60b9add
  Built:            Thu Apr 11 10:55:26 2024
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.6.31
  GitCommit:        e377cd56a71523140ca6ae87e30244719194a521
 runc:
  Version:          1.1.12
  GitCommit:        v1.1.12-0-g51d5e94
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```

## CentOS 7 安装 docker-compose

Linux 上我们可以从 Github 上下载它的二进制包来使用，最新发行的版本地址：https://github.com/docker/compose/releases。

```
# step 1: 运行以下命令以下载 Docker Compose 的当前稳定版本，要安装其他版本的 Compose，请替换 v2.26.1。
sudo curl -L "https://github.com/docker/compose/releases/download/v2.26.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# step 2: 将可执行权限应用于二进制文件：
sudo chmod +x /usr/local/bin/docker-compose

# step 3: 创建软链：
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

# step 4: 测试是否安装成功：
docker-compose --version
```

if [[ ${NGINX_ENABLE} == 'true' ]]; then
  cat >> ./docker-compose.yaml <<EOF
  nginx:
    # container_name: ${ENV}-nginx
    build:
      context: ./nginx
      args:
        - CHANGE_SOURCE=${CHANGE_SOURCE}
        # - PHP_UPSTREAM_CONTAINER=${NGINX_PHP_UPSTREAM_CONTAINER}
        # - PHP_UPSTREAM_PORT=${NGINX_PHP_UPSTREAM_PORT}
        - http_proxy
        - https_proxy
        - no_proxy
    volumes:
      - ./nginx/logs/:/var/log
      - ./nginx/sites/:/etc/nginx/sites-available
      - ./nginx/ssl/:/etc/nginx/ssl
      # - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ${WORKSPACE}/www:/var/www/site
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

EOF
fi

if 里面根据PHP_ENABLE的值判断是否是TRUE，如果是 TRUE才在tty下面输出depends_on


alertmanager 是否存在指纹相同，告警时间不同的告警记录


// 如果警报未解决，则 aggrGroup.flush(https://github.com/prometheus/alertmanager/pull/1686) 中的endsAt 将被设置为0
// 因此，如果 24 小时后仍未收到，请将其设置为自动解决。 
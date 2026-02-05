+++
title = 'Docker安装与配置'
date = 2026-01-21T09:23:56+08:00 
draft = true
categories = [ "Docker" ]
tags = [ "docker" ]
+++

<!--more-->

本文记录 2026 年初仍然有效的 Docker Engine（社区版）及 Docker Compose v2 的安装方式，适用于大多数现代 Linux 发行版。

**适用系统**：CentOS 8/Stream、Rocky/AlmaLinux 8/9、Ubuntu 20.04/22.04/24.04、Debian 11/12 等

**主要内容**：
- 卸载旧版 → 安装最新 Docker CE
- 配置国内镜像加速
- 安装 Docker Compose v2（官方推荐的独立二进制方式）
- 验证安装 & 常用配置建议

> **重要**：操作前建议先卸载系统自带的旧 docker 包，避免冲突。

## 一、安装 Docker Engine（CE）

### 1. 参考文档按步骤操作
- [Docker CE镜像](https://developer.aliyun.com/mirror/docker-ce?spm=a2c6h.13651102.0.0.74c71b11rDkV9p)


### 2. 启动服务并设置为开机自启

```bash
sudo service docker start
sudo systemctl enable docker
```

### 3. 配置国内镜像加速（强烈建议）

编辑（或创建）`/etc/docker/daemon.json`：

```json
{
  "registry-mirrors": [
    "https://docker.m.daocloud.io",
    "https://mirror.ccs.tencentyun.com",
    "https://docker.mirrors.ustc.edu.cn",
    "https://registry.docker-cn.com",
    "https://vfs1y0dl.mirror.aliyuncs.com"
  ],
  "exec-opts": ["native.cgroupdriver=systemd"]
}
```

使配置生效：

```bash
sudo systemctl daemon-reload
sudo service docker restart
```

验证镜像配置是否生效：

```bash
docker info --format '{{json .RegistryConfig.Mirrors}}' | jq
```

## 二、安装 Docker Compose v2（推荐独立二进制方式）

> 2023 年之后官方已停止维护旧的 Python 版 `docker-compose`，全面转向 Compose V2（新命令：`docker compose`）

### 1. 参考文档按步骤操作
- [Docker Compose安装](https://cloud.tencent.com/developer/article/1942706?cps_key=1d358d18a7a17b4a6df8d67a62fd3d3d)
- [Docker Compose release](https://github.com/docker/compose/releases)

### 2 步骤
```bash
# step 1: 运行以下命令以下载 Docker Compose 的当前稳定版本，要安装其他版本的 Compose，请替换 v2.28.0。
sudo curl -L "https://github.com/docker/compose/releases/download/v2.28.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# step 2: 将可执行权限应用于二进制文件：
sudo chmod +x /usr/local/bin/docker-compose

# step 3: 创建软链：
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

# step 4: 测试是否安装成功：
docker-compose --version
```


+++
title = 'Mac彻底卸载Docker'
date = 2024-10-26T10:58:11+08:00
draft = false
categories = [ "Docker" ]
tags = [ "docker" ]
+++

# 一 介绍

Mac 上彻底卸载 Docker 包括一些阶段：

- 手动删除 Docker 应用程序
- 清理文件和配置
- 移除创建的容器和网络

# 二 移除容器网络设置

```bash
docker system prune -a
```

# 三 删除应用程序

## 1、通过Finder删除Docker

这是最直接的方法。操作如下：

1. 打开Finder。
2. 进入“应用程序”文件夹。
3. 找到Docker应用程序。
4. 将Docker拖动到废纸篓，或者右键点击选择“移到废纸篓”。

## 2、使用命令行工具删除Docker

这种方式将强制删除Docker应用程序。打开终端，输入命令：
```bash
sudo rm -rf /Applications/Docker.app
```

# 四 清理文件和配置

## 1、删除配置文件和数据

Docker在你的系统中会创建许多配置文件和数据文件，这些文件通常位于你的用户目录下。使用以下命令逐一删除这些文件：
```bash
sudo rm -rf ~/Library/Containers/com.docker.docker
sudo rm -rf ~/Library/Application Support/Docker Desktop
sudo rm -rf ~/Library/Group Containers/group.com.docker
sudo rm -rf ~/.docker
```

## 2、清理系统缓存和日志

Docker还会在系统缓存和日志中留下许多文件。你可以通过以下命令来清理这些文件：
```bash
sudo rm -rf ~/Library/Caches/com.docker.docker
sudo rm -rf ~/Library/Logs/Docker Desktop
```

# 五 重启系统

重启系统，以确保所有与Docker相关的进程和文件都被彻底清除。

# 六 注意

1. 备份重要数据。
2. 检查系统完整性。

# 七 小结

将上面清理命令整理为shell脚本，一键清理Docker:
```bash
#!/bin/bash

# 检查 figlet 是否安装
if command -v figlet &> /dev/null; then
    figlet "Uninstall Docker"
fi

# 一、移除容器网络设置
docker system prune -a

# 二、删除应用程序

# 1、使用命令行工具删除Docker
echo "删除 Docker 应用程序..."
sudo rm -rf /Applications/Docker.app

# 三、清理文件和配置

# 1、删除配置文件和数据
echo "删除配置文件和数据..."
sudo rm -rf ~/Library/Containers/com.docker.docker
sudo rm -rf ~/Library/Application\ Support/Docker\ Desktop
sudo rm -rf ~/Library/Group\ Containers/group.com.docker
sudo rm -rf ~/.docker

# 2、清理系统缓存和日志
echo "清理系统缓存和日志..."
sudo rm -rf ~/Library/Caches/com.docker.docker
sudo rm -rf ~/Library/Logs/Docker\ Desktop

# 四、提示重启系统
echo "请重启系统以确保所有与 Docker 相关的进程和文件都被彻底清除。"
```

# 七 参考

[Mac上如何彻底卸载docker](https://docs.pingcode.com/baike/3475080)
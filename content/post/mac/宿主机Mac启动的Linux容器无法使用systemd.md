+++
title = '宿主机Mac启动的Linux容器无法使用systemd'
date = 2024-11-02T10:16:28+08:00
draft = false
categories = [ "Mac" ]
tag = [ "mac", "docker" ]
+++

## 背景

- 宿主机：Mac
- 容器：在Mac上安装的 CentOS8 容器
- 现象：

  1、`docker-comose up` 启动容器，提示信息如下：
  ```bash
  vm-1   |
  vm-1   | Welcome to CentOS Linux 8!
  vm-1   |
  vm-1   | [!!!!!!] Failed to allocate manager object, freezing.
  ```

  2、进入容器中，无法使用 `systemctl` 命令，提示信息如下：
  ```bash
  [root@vm-1 opt]# systemctl
  Failed to connect to bus: No such file or directory
  ```

## 原因

**参考**

- [Docker Desktop 4.3.0 release-notes](https://docs.docker.com/desktop/release-notes/#430)
- [Mac Docker下安装centos7报错Failed to get D-Bus connection: No such file or directory完美解决方案](https://blog.csdn.net/lideqiang110119/article/details/129525432)
- [Mac M1使用Docker报错 Failed to get D-Bus connection: No such file or directory的解决方案](https://blog.csdn.net/counsellor/article/details/128448999)

**原因**

系统不兼容导致。

## 解决方案

**参考**

- [Docker Desktop 4.34及早期版本为settings.json](https://docs.docker.com/extensions/settings-feedback/)

**我的**
1. sudo vim ~/Library/Group\ Containers/group.com.docker/settings-store.json。
2. 修改 `DeprecatedCgroupv1` 参数为 `true`，默认是 false
3. 然后重启 Docker 环境

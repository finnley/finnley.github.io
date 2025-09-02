+++
title = 'Vagrant介绍'
date = 2018-12-16T13:51:53+08:00
draft = true
categories = [ "Vagrant" ]
tags = [ "vagrant" ]
+++

# Virtualization

虚拟机技术里面非常重要的一个组件 `Hypervisor`，由它来实现底层物理资源的虚拟化，在它之上才可以新建一些虚拟机，根据它的位置定位可以分为两类。
一类（Type 1）可以 **直接在物理服务器** 上安装，安装完之后可以在它之上创建 Linux 或 Windows 虚拟机了，这种直接在物理机上安装，物理机是没有安装任何操作系统的物理机
二类（Type 2）是需要在物理机上先安装某个操作系统（OS），`Hypervisor` 是安装在这个 OS 之上，安装完之后再在它之上创建 Linux 或 Windows 虚拟机

一类（Type 1）的 `Hypervisor` 是可以直接和物理机交互的，二类（Type 2）需要经过 OS 这一层，然后才能和物理机交互，从效率上看，肯定是 一类（Type 1）的效率高，所以在生产环境下一般使用功能的一类，但是对于我们的个人电脑使用功能一类是不太合适的，而是使用的二类，比如VirtialBox，VMWare

# 什么是Vagrant

`Vagrant` 是构建在 `虚拟化技术之上` 的 `虚拟机运行环境管理工具`

  * 可以建立和删除虚拟机
  * 可以配置虚拟机的运行参数(如CPU，内存)
  * 管理虚拟机运行状态（如启动虚拟机，重新启动虚拟机，关闭虚拟机，将虚拟机挂起等）
  * 自动化配置和安装开发环境
  * 可以打包和分发虚拟机运行环境

`Vagrant` 的运行，需要依赖某项具体的 `虚拟化技术`

  * VirtualBox
  * WMWare

## 使用Vagran好处

1. 个人角度

  * 跨平台
  * 可移动
  * 自动化部署无需人工参与（可以自动帮我们启动虚拟机或执行一些脚本安装软件）

2. 公司角度

  * 减少人力培训成本
  * 统一开发环境（可以保证本地开发环境和线上测试环境一样）

# Vagrant适用范围

* 开发环境（线上环境可以使用docker）
* 项目配置比较复杂

# How vagrant works?

![](https://images.notes.einscat.com/vagrant/116.png)

vagrant 作为一个工具，本身是不可能创建虚拟机的，必须依赖于底层的Hypervisor，vagrant 可以通过不同的 Provider 来创建对应的虚拟机，Provider就是和不同虚拟机创建的Hypervisor

vagrant 会基于 Box 基础的API去调用 Provider 的API，然后在 Hypervisor 上去创建对应的虚拟机

# 前提准备

安装 `VirtualBox`
安装 `Vagrant`

# 常用命令

虚拟机本质底层仍然是操作系统，通常虚拟机安装系统一般是以ISO镜像来安装，而vagrant默认的格式是以 `.box` 结尾的。

## 查看目前已有的box

```
vagrant box list 
```

## 新添加一个box

```
vagrant box add <box名称>
```

## 删除指定box

```
vagrant box remove <box名称>
```

![](https://images.notes.xuepincat.com/vagrant/1.png)

## 初始化配置vagrantfile配置文件

* 启动虚拟机的配置文件

```
vagrant init
```

## 启动虚拟机

* 如果当前没有则创建并启动一个新的虚拟机

```
vagrant up
```

## ssh登录虚拟机

```
vagrant ssh
```

## 挂起虚拟机

* 不会消耗任何内存或者硬件资源

```
vagrant suspend
```

## 重启虚拟机

```
vagrant reload
```

## 关闭虚拟机

```
vagrant halt
```

## 查看虚拟机状态

```
vagrant status
```

## 删除虚拟机

```
vagrant destroy
```

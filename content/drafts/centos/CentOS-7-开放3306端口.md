+++
title = 'CentOS 7 开放3306端口'
date = 2018-07-20T14:20:15+08:00
draft = true
categories = [ "CentOS" ]
tags = [ "centos" ]
+++

# 问题描述

我准备了两台虚拟机分别安装 `CentOS 7`，一台IP `192.168.1.151`，一台IP `192.168.1.152`，安装完 `MySQL` 后使用 `Navicat` 连接的时候出现了下面错误提示

![](/images/centos/20180720/1-1.png)

![](/images/centos/20180720/1-2.png)

后来检查发现是因为 `CentOS` 没有开放 `3306` 端口。在 `Centos 7` 或 `RHEL 7` 或 `Fedora` 中防火墙由 `firewalld` 来管理，而不是 `iptables`

现在我将在151的机器上使用firewalld,152的机器上我使用iptables

# `firewalld` 防火墙

## 语法

```
firewall-cmd [--zone=<zone>] --add-port=<port>[-<port>]/<protocol> [--timeout=<seconds>]
```

该命令表示启用端口和协议的组合，端口可以是一个单独的端口 <port> 或者是一个端口范围 <port>-<port>，协议可以是 `tcp` 或 `udp`

## `firewalld` 常用命令

### 查看版本

```
firewall-cmd --version
```

![](/images/centos/20180720/2.png)

### 查看状态

```
firewall-cmd --state
```

![](/images/centos/20180720/3.png)

或者

```
systemctl status firewalld
```

![](/images/centos/20180720/4.png)

或者

```
service firewalld status
```

![](/images/centos/20180720/5.png)



### 查看所有打开的端口

```
firewall-cmd --zone=public --list-ports
```

![](/images/centos/20180720/6.png)

### 查看指定端口是否开启

* 查询端口号 80 是否开启

```
firewall-cmd --query-port=80/tcp
```

* 查询端口号 3306 是否开启

```
firewall-cmd --query-port=3306/tcp
```

![](/images/centos/20180720/7.png)

### 开放指定端口

如永久开放80端口号

```
firewall-cmd --permanent --zone=public --add-port=80/tcp
```

### 重载防火墙

上面命令设置后需要重载防火墙

```
firewall-cmd --reload
```

`--zone`: 作用域
`--add-port=80/tcp`: 添加端口，格式为：端口/通讯协议
`--permanent`: 永久生效，没有此参数重启后失效

1. 永久开放3306端口号

```
firewall-cmd --permanent --zone=public --add-port=3306/tcp
```

![](/images/centos/20180720/8.png)

2. 重启 `firewalld`

上面开放了端口之后还需要重启防火墙使其生效

```
systemctl restart firewalld
```

3. 重启之后查看一下开放端口情况

```
firewall-cmd --zone=public --list-ports
```

![](/images/centos/20180720/9.png)

### 开启和关闭 `firewalld`

```
systemctl start firewalld
systemctl stop firewalld
```

到这里我们在使用Navicat测试一下是否可以连接MySQL

![](/images/centos/20180720/10.png)

# `iptables` 防火墙

上面提到Centos 7中防火墙由 firewalld 来管理，而不是 iptables，CentOS 7默认使用的是 `firewall` 作为防火墙，这里我使用152的机子改为 `iptables` 防火墙

* 关闭 `firewalld`

将 `CentOS` 默认的 `firewalld` 改为 `iptables` 之前需要将系统默认的 `firewalld` 关闭

```
systemctl stop firewalld.service
systemctl disable firewalld.service
systemctl mask firewalld.service #屏蔽服务（让它不能启动）
```

![](/images/centos/20180720/11.png)

* 安装 `iptables`

这时候如果开启 `iptables` ，会发现提示 `Failed to start iptables.service: Unit not found.` 信息，表示机子上还没有安装 `iptables`

![](/images/centos/20180720/12.png)

使用下面命令安装 `iptables`

```
yum install -y iptables-services
```

![](/images/centos/20180720/13.png)

* 启动 `iptables`

```
systemctl start iptables
systemctl enable iptables #设置防火墙开机启动
```

![](/images/centos/20180720/14.png)

* 查看防火墙状态

```
systemctl status iptables
```

![](/images/centos/20180720/15.png)

* 查看开放端口

netstat -nupl (UDP类型的端口)
netstat -ntpl (TCP类型的端口)


> a 表示所有
> n表示不查询dns
> t表示tcp协议
> u表示udp协议
> p表示查询占用的程序
> l表示查询正在监听的程序
> netstat -nuplf|grep 3306: 这个表示查找处于监听状态的，端口号为3306的进程

![](/images/centos/20180720/16.png)

* 编辑防火墙，增加端口

如：添加80端口和3306端口

```
vi /etc/sysconfig/iptables #编辑防火墙配置文件
```

添加下面两句话

```
-A INPUT -p tcp -m state --state NEW -m tcp --dport 80 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 3306 -j ACCEPT
```

![](/images/centos/20180720/17.png)

* 重启配置

```
systemctl restart iptables.service #重启防火墙使配置生效
systemctl enable iptables.service 
```

![](/images/centos/20180720/18.png)

我再去使用 Navticat 连接下 MySQL 数据库，看是否连接成功

![](/images/centos/20180720/19.png)

成功。


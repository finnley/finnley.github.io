+++
title = 'CentOS配置Node环境'
date = 2019-04-07T18:01:24+08:00
draft = true
categories = [ "CentOS" ]
tags = [ "centos" ]
+++

## 准备
安装前检查系统是否已经安装了Node环境

```
node -v
npm -v
```

![](/img/node/1.png)

<!-- more -->

## 下载

```
wget https://nodejs.org/dist/v10.15.3/node-v10.15.3-linux-x64.tar.xz
```

![](/img/node/2.png)

![](/img/node/3.png)

## 解压

解压tar.xz文件：先 xz -d xxx.tar.xz 将 xxx.tar.xz解压成 xxx.tar 然后，再用 tar xvf xxx.tar来解包

```
xz -d node-v10.15.3-linux-x64.tar.xz
tar xvf node-v10.15.3-linux-x64.tar
```

![](/img/node/4.png)

## 配置

进入解压后的目录中的 `bin` 目录

```
cd node-v10.15.3-linux-x64/bin/
ls -l
```

![](/img/node/5.png)

会看到终端会处于 `node` 和 `npm` 的相关信息

使用下面命令进行配置

```
cd ~
ln -s /root/node-v10.15.3-linux-x64/bin/node /usr/local/bin/node
 ln -s /root/node-v10.15.3-linux-x64/bin/npm /usr/local/bin/npm
```

![](/img/node/6.png)

使用下面命令查看版本

```
node -v
npm -v
``

![](/img/node/7.png)

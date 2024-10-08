+++
title = 'Ubuntu16.04安装Navicat'
date = 2017-06-07T15:39:46+08:00
draft = true
categories = [ "Ubuntu" ]
tags = [ "ubuntu" ]
+++

# 下载

![](/images/ubuntu/mysql/1.png)

# 解压

``` bash
$ sudo tar zxvf navicat112_premium_cs_x64.tar.gz
```

# 重命名

``` bash
$ sudo mv navicat112_premium_cs_x64 navicat
```

![](/images/ubuntu/mysql/3.png)

# 启动

``` bash
$ cd navicat
$ sudo ./start_navicat
```

启动后界面如下图所示：

![](/images/ubuntu/mysql/4.png)

点击启动界面左边的按钮，进入软件主页面

![](/images/ubuntu/mysql/5.png)

这时候会发现软件界面都是乱码，这就是下面我需要解决的问题。

# 解决软件乱码问题

打开 `start_navicat` 文件，会看到 `export LANG="en_US.UTF-8"` 将这句话改为 `export LANG="zh_CN.UTF-8"`

``` bash
$ sudo gedit start_navicat
```

找到 `export LANG="en_US.UTF-8"` 这句话并进行修改

修改前：

``` bash
export LANG="en_US.UTF-8"
```

修改后：

``` bash
export LANG="zh_CN.UTF-8"
```

再次启动，会发现已经没有乱码了：

![](/images/ubuntu/mysql/6.png)

![](/images/ubuntu/mysql/7.png)

# 创建桌面快捷方式

首先需要一个 `navicat` 的图标，解压后的 navicat 中没有图标，所以我到网上找了张图标，我把它放在 `/opt/navicat/Navicat` 目录下

![](/images/ubuntu/mysql/navicat-premium-logo.png)

切换到 `/usr/share/applications` 目录下

``` bash
$ cd /usr/share/applications
```

![](/images/ubuntu/mysql/8.png)

创建 `navicat.desktop` 文件

``` bash
$ sudo gedit navicat.desktop
```

内容如下：

``` bash
[Desktop Entry]
Encoding=UTF-8
Name=Navicat
Comment=Navicat Premium
Exec=/opt/navicat/start_navicat
Icon=/opt/navicat/Navicat/navicat.png
Terminal=false
StartupNotify=true
Type=Application
Categories=Application;Development;
```

![](/images/ubuntu/mysql/10.png)

这时候到 `/usr/share/applications` 目录下去查看，会发现多了navicat的图标

![](/images/ubuntu/mysql/11.png)

将该图标复制到桌面上，这时候桌面就会出现桌面快捷图标，就不用每次启动的时候都要进入安装包使用命令启动了。

![](/images/ubuntu/mysql/12.png)

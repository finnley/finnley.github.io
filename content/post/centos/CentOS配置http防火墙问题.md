+++
title = 'CentOS配置http防火墙问题'
date = 2017-12-29T23:43:07+08:00
draft = true
categories = [ "CentOS" ]
tags = [ "centos" ]
+++

![](/images/centos/0.jpg)

CentoS下安装完http需要配置httpd防火墙，在配置的过程中出现了下面的问题，下面是错误重现以及解决方法。

在执行 `firewall-cmd --zone=public --permanent --add-service=http` 提示 `FirewallD is not running`

![](/images/centos/firewall/2.png)

这时我们可以通过下面命令查看 `firewalld` 状态，发现当前是 `dead` 状态，表示防火墙未开启。

``` bash
# systemctl status firewalld
```

![](/images/centos/firewall/3.png)

使用下面命令开启防火墙，没有任何提示即开启成功。

``` bash
# systemctl start firewalld
```

![](/images/centos/firewall/4.png)

再通过下面命令查看 firewalld 状态，显示 `running`，颜色也由灰色转为绿色即表示已开启了。

``` bash
# systemctl status firewalld
```

![](/images/centos/firewall/5.png)

如果要关闭防火墙设置，可能通过下面命令这条指令来关闭该功能。

``` bash
# systemctl stop firewalld
```
现在我们再次执行执行上面的命令，提示 `success`，表示设置成功，这样就可以继续后面的设置了。

对于最开始出现的 `FirewallD is not running` 错误可以通过执行下面指令解决

``` bash
# firewall-cmd --zone=public --permanent --add-service=http
# firewall-cmd --zone=public --permanent --add-service=https
# firewall-cmd --reload
```
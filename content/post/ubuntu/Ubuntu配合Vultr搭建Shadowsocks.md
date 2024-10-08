+++
title = 'Ubuntu配合Vultr搭建Shadowsocks'
date = 2018-09-09T00:31:12+08:00
draft = true
categories = [ "Ubuntu" ]
tags = [ "ubuntu" ]
+++

## 环境安装

```
sudo apt-get update
sudo apt-get install -y python-pip python-setuptools m2crypto
pip install shadowsocks
```

![](/images/ubuntu/shadowsocks/1.png)

最后一行安装 `shadowsocks` 时候可能会警告提示，不用理会，只要最后一行有shadowsocks2.8.2（目前最新版本）出现就ok了

## 配置

```
nano ss.sh
```

复制下面的脚本代码

```
#! /bin/sh

#按照提示来输入配置参数
setup()
{
    echo "输入你的服务器的IP地址："
    read server
    echo "输入想设定的服务器接口（推荐大于2000的四位数或五位数如9400）："
    read server_port
    echo "输入你要设置的密码："
    read passwd

    echo
    echo
    echo "配置如下："
    echo "ip          :${server}"
    echo "password    :${passwd}"
    echo "port        :${server_port}"
    echo "metho       :aes-256-cfb"
    echo
    echo
}
setup
while :
do
    echo "确定配置信息吗[Y/n]\c"
    read sure
    if [ ${sure:=y} = "n" ] || [ ${sure:=y} = "N" ]; then
        setup
    else
        break
    fi
done

#生成json文件
config="/etc/shadowsocks.json"
if [ -f "${config}" ]; then
    `rm $config`
fi
`touch $config`
`echo { >> $config`
`echo "\"server\":\"${server}\"," >> $config`
`echo "\"server_port\":${server_port}," >> $config`
`echo "\"local_port\":1080," >> $config`
`echo "\"password\":\"${passwd}\"," >> $config`
`echo "\"timeout\":300," >> $config`
`echo "\"method\":\"aes-256-cfb\"" >> $config`
`echo } >> $config`

echo `ssserver -c "${config}" -d start`
#开启BBR拥塞算法，提高速度
sysctl net.core.default_qdisc=fq
sysctl net.ipv4.tcp_congestion_control=bbr
lsmod | grep bbr
```

按Crl+X，然后按Y，然后按回车，进行保存文件并退出。



在运行这个脚本之前，要改个文件里的两处代码（目前shadowsocks2.8.2这个版本需要改，后期更新后可能会修正） 
输入：

```
sudo vi /usr/local/lib/python2.7/dist-packages/shadowsocks/crypto/openssl.py
```

输入 `/cleanup` 搜索 `cleanup` ,将 `cleanup` 修改为 `reset` ,总共有两个地方要修改，最后保存退出。

修改之后输入

```
sudo chmod 755 ss.sh
./ss.sh
```

![](/images/ubuntu/shadowsocks/2.png)

运行脚本，按照提示来输入你要配置的信息，确定信息后就完成了。 

## 下载

[shadowsocks](https://shadowsocks.org/en/index.html) 

下载好后解压到目录双击打开安装包，输入刚才配置的内容

![](/images/ubuntu/shadowsocks/3.png)

配置完后设置开机启动，使用PAC模式，也就是自动模式

![](/images/ubuntu/shadowsocks/4.png)

打开Youtube视频还是很流畅的

![](/images/ubuntu/shadowsocks/5.png)


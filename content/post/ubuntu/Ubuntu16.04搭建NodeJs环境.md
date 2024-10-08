+++
title = 'Ubuntu16.04搭建NodeJs环境'
date = 2017-06-24T17:40:07+08:00
draft = true
categories = [ "Ubuntu" ]
tags = [ "ubuntu" ]
+++

# 下载

当前最新版本的 node 是 v6.11.0

![](/images/ubuntu/nodejs/1.png)

![](/images/ubuntu/nodejs/2.png)

# 解压

``` bash
$ sudo tar xvf node-v6.11.0-linux-x64.tar.xz
```

解压完之后得到了这样的文件夹:

![](/images/ubuntu/nodejs/4.png)


# 对文件夹重命名

``` bash
$ sudo mv node-v6.11.0-linux-x64 node
```

# 赋权

``` bash
$ sudo chmod 777 /opt/node -R
```

# 配置

* 设置 nodejs 软链

``` bash
$ sudo ln -s /opt/node/bin/node /usr/local/bin/node
```

* 设置 npm 软链

``` bash
$ sudo ln -s /opt/node/bin/npm /usr/local/bin/npm
```

![](/images/ubuntu/nodejs/6.png)


配置前：

![](/images/ubuntu/nodejs/7.png)

配置后：

![](/images/ubuntu/nodejs/8.png)


# 添加环境变量

打开终端在终端中输入下面命令：

``` bash
$ sudo gedit ~/.bashrc
```

打开文件后，在文件末尾加上下面一段：

``` bash
export PATH=$PATH:/opt/node/bin
```

使其立即生效

``` bash
$ source ~/.bashrc
```

# 验证

``` bash
$ node -v
$ npm -v
```

![](/images/ubuntu/nodejs/9.png)

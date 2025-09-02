+++
title = 'Ubuntu16.04安装最新版本Git'
date = 2017-12-30T15:53:14+08:00
draft = true
categories = [ "Ubuntu" ]
tags = [ "ubuntu" ]
+++

当前最新版本Git是2.15.1

![](/images/ubuntu/git/1.png)

添加源

``` bash
$ sudo add-apt-repository ppa:git-core/ppa
$ sudo apt update
```

![](/images/ubuntu/git/2.png)
![](/images/ubuntu/git/3.png)

安装

``` bash
$ sudo apt -y install git
```

![](/images/ubuntu/git/4.png)

查看版本

```bash
$ git –-version
```

![](/images/ubuntu/git/5.png)

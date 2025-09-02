+++
title = 'Hexo+NexT基础设置'
date = 2017-03-30T21:33:05+08:00
expiryDate = 2024-12-31T23:59:59Z
draft = true
categories = [ "Hexo" ]
tags = [ "hexo" ]
+++

# 背景

以前看到别人有自己的博客很是羡慕，也想有一个属于自己的博客，但是肚子里又没有墨水，生怕没东西写（自卑，不够自信，嗯，天生的...），就算能扯点，也担心不能坚持下去。

后来...
嗯，真香（虚荣心作怪，积累了几篇膨胀了...）

(切😒 ，还好意思，写了几年，还都是傻瓜式的流水笔记，毫无深度😒)

我们浏览招聘网站的时候，往往会看到有些公司招聘时会要求附上自己的作品，或者是有自己的Github，或者是自己的技术博客等。作为刚毕业的萌新来说，希望在投递简历面试的时候增加一些通过的几率。

<img src="http://images.notes.xuepincat.com/hexo/git/1.png" height="550" width="300">

这篇就是记录如何使用 `GitHub` 配合 `Hexo` 和 `NexT` 搭建自己博客的。在搭建博客的时候还需要做一些准备，比如需要安装 `Git`, `NodeJS` 等。

# 什么是Hexo?

简单的概括，[`Hexo`](https://hexo.io/) 是一个快速、简洁且高效的基于 `Node.js` 的静态博客框架。

# 安装并初始化

[`Hexo官方文档`](https://hexo.io/zh-cn/docs/)

```
$ npm install hexo-cli -g
$ hexo init blog
$ cd blog
$ npm install
$ hexo server
```

* 安装

``` bash
npm install -g hexo-cli
```

!["hexo"](/images/hexo/hexo/1.png)

!["hexo"](/images/hexo/hexo/2.png)

在结尾出现这样的信息，属于正常现象，不必担心。

安装完后会在 `node_global` 下的 `node_modules` 文件夹下会出现下面的内容

!["hexo"](/images/hexo/hexo/3.png)

* 初始化

创建一个目标文件夹，用于放置 `Hexo`。比如我在 `G` 盘下的 `GitHub` 文件夹中创建了一个名为 `Hexo` 的目录

!["hexo"](/images/hexo/hexo/4.png)

进入该目录，快捷键 `SHIFT + 鼠标右键` ，召唤出命令行窗口

``` bash
hexo init
```

!["hexo"](/images/hexo/hexo/5.png)

我这边使用的 `cd` 命令切换到 `Hexo` 文件夹下的，回车之后会产生一堆信息，完了之后在 `Hexo` 文件夹中会多出一些文件

!["hexo"](/images/hexo/hexo/6.png)

当然也可以直接使用 `hexo init Hexo` 命令进行初始化。

在这些文件夹中 `themes` 是主题文件夹，Hexo会根据主题来生成静态页面。

至于文件里面有什么东西，我暂且没有去管它们，先把效果做出来，日后再去看里面的东西(大概率是不会看了，也看不懂，主要还是因为懒)。

``` bash
cd Hexo
npm install
```

* 启动测试

``` bash
hexo server
```

或者

``` bash
hexo s
```
输入完命令会提示这样的信息，根据提示在本地浏览器地址栏中输入 `localhost:4000` ，退出按快捷键 `CTRL + C`

!["hexo"](/images/hexo/hexo/7.png)

如果出现这样的页面，则表示 Hexo 安装成功。

!["hexo"](/images/hexo/hexo/8.png)


**注意**

> 在输入 `localhost:4000` 进行测试的时候，端口号是 `4000` ，有一次我在输入这个进行测试的时候就没有能够成功显示上面的页面，这时候可以检查 `4000` 端口是否被占用了，如果被占用了，页面就不会显示。

> 我的端口被占用了的是因为我安装了 `foxreader` ，`foxreader阅读器` 进程没有杀掉。解决办法可以输入下面命令查看被占用端口情况，然后杀死进程。当然这些是题外话了。
``` bash
netstat -ano
```

# 生成静态文件

``` bash
hexo generate
```

或者

``` bash
hexo g
```

# 上传到站点

该命令需要设置上传的仓库，后面介绍，这里只是介绍下有这个命令，

``` bash
hexo deploy
```

或者

``` bash
hexo d
```

到这里Hexo基本安装完了。

PS: 今天写的东西有点少，就像写作文凑个字数，在下一篇中将介绍 `NexT` 主题的下载与配置，以及如何将它们上传到 `GitHub`，然后就可以看到博客的雏形了，想想就觉的激动。


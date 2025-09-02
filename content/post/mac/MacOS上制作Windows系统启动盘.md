+++
title = 'MacOS上制作Windows系统启动盘'
date = 2025-01-29T12:02:13+08:00
draft = false
categories = [ "Mac" ]
tag = [ "mac", "windows" ]
+++

年前买了一些电脑配配件，今天2025年正月初一在家装系统。

步骤如下：

1、 准备容量大于 8G 的 U盘。

2、插入U盘，打开 Mac 上的 `Disk Utility`。

3、点击 【Erase】抹除 U盘，注意 `Format` 选择 `ExFAT`。

![](/images/mac/windows/10.png)

4、下载 Window 11 ISO 镜像，比如文件名为 `Win11_24H2_Chinese_Simplified_x64.iso`。

- [win1123h2下载链接在哪儿，目前在官网上下载的win11都是24h2](https://answers.microsoft.com/zh-hans/windows/forum/all/win1123h2%E4%B8%8B%E8%BD%BD%E9%93%BE%E6%8E%A5/9421c1eb-04bf-46ad-8e17-1da842e4d522)
- [多版本](https://msdl.gravesoft.dev/#2860)
- [家庭中文版](https://msdl.gravesoft.dev/#2861)


5、双击挂载 ISO 镜像。

![](/images/mac/windows/20.png)

6、打开终端执行下面命令刻录Windows系统到U盘，然后等待完成。

    ```bash
    cp -rp /Volumes/CCCOMA_X64FRE_ZH-CN_DV9/* /Volumes/WINDOW11
    ```
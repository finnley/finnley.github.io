+++
title = 'MacOS上制作Windows/Ubuntu系统启动盘'
date = 2025-01-29T12:02:13+08:00
draft = false
categories = [ "Mac" ]
tag = [ "mac", "windows", "ubuntu" ]
+++

# 1 Windows 11

年前买了一些电脑配配件，今天2025年正月初一在家装系统。

## 1.1 准备

- 容量大于 `8G` 的 U盘。
- Window 11 ISO 镜像。
    - [多版本](https://msdl.gravesoft.dev/#2860)
    - [家庭中文版](https://msdl.gravesoft.dev/#2861)

## 1.2 步骤

1、 、插入U盘，打开 Mac 上的 `Disk Utility`。

2、点击 【Erase】抹除 U盘，注意 `Format` 选择 `ExFAT`。

![](/images/mac/10.png)

3、下载 Window 11 ISO 镜像，比如文件名为 `Win11_24H2_Chinese_Simplified_x64.iso`。

- [多版本](https://msdl.gravesoft.dev/#2860)
- [家庭中文版](https://msdl.gravesoft.dev/#2861)

4、双击挂载 ISO 镜像。

![](/images/mac/20.png)

5、打开终端执行下面命令刻录Windows系统到U盘，然后等待完成。

    ```bash
    cp -rp /Volumes/CCCOMA_X64FRE_ZH-CN_DV9/* /Volumes/WINDOW11
    ```

参考：[win1123h2下载链接在哪儿，目前在官网上下载的win11都是24h2](https://learn.microsoft.com/zh-cn/answers/questions/3936555/win1123h2-win11-24h2?forum=windows-all&referrer=answers)


# 2 Ubuntu24.04

## 2.1 准备

- Ubuntu ISO 镜像
- 至少 `4G` 容量的 U盘
- 下砸 [balenaEtcher](https://etcher.balena.io/)

## 2.2 步骤

1、插入U盘，打开 Mac 上的 `Disk Utility`。

2、选择你的U盘，点击 【Erase】抹除 U盘，注意 `Format` 选择 `MS-DOS (FAT)`。

3、打开 balenaEtcher，选择Ubuntu镜像与刻录媒介，然后点击【Flash】刻录。

![](/images/mac/30.png)

参考：

- [Create a bootable USB stick on macOS](https://ubuntu.com/tutorials/create-a-usb-stick-on-macos#1-overview)
- [Mac下如何制作Ubuntu系统的USB启动盘](https://cto.eguidedog.net/node/826)


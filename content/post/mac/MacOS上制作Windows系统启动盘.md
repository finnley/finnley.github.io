+++
title = 'MacOS上制作Windows系统启动盘'
date = 2025-01-29T12:02:13+08:00
draft = true
categories = [ "Mac" ]
tag = [ "mac", "windows" ]
+++

1、插入 U盘。

2、打开 Mac 上的 `Disk Utility`。

3、点击 【Erase】抹除 U盘，注意 `Format` 选择 `ExFAT`。

![](/images/mac/windows/10.png)

4、下载 Window 11 ISO 镜像，比如文件名为 `Win11_24H2_Chinese_Simplified_x64.iso`。

5、双击挂载 ISO 镜像。

![](/images/mac/windows/20.png)

6、打开终端执行下面命令刻录Windows系统到U盘，然后等待完成。

```bash
cp -rp /Volumes/CCCOMA_X64FRE_ZH-CN_DV9/* /Volumes/WINDOW11
```
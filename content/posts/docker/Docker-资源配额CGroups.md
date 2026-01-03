+++
title = 'Docker-资源配额CGroups'
date = 2020-04-20T21:03:53+08:00
draft = true
categories = [ "Docker" ]
tags = [ "docker" ]
+++

# 

Docker 以应用的形式安装在宿主机上，如果进程过多，势必会出现进程抢占的情况。比如内存，如果我们不加以限制，内存的占用将会越来越多，针对这个现象，docker需要解决的一个问题就是资源如何分配的问题。
也就是容器可以安装在宿主机上


[FAILED] Failed unmounting cdrom.mount - /cdrom.
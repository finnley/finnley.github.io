+++
title = '宿主机Mac无法Ping通容器IP'
date = 2025-01-05T22:36:14+08:00
draft = false
categories = [ "Mac" ]
tag = [ "mac", "docker" ]
+++

1、 进入虚拟机
```shell
docker run --pid=host --privileged -it --rm justincormack/nsenter1
```

2、进入目录
```shell
cd containers/services/docker/rootfs
```


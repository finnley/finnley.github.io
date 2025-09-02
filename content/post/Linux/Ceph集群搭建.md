+++
title = 'Ceph集群搭建'
date = 2024-08-10T12:43:35+08:00
draft = false
categories = [ "Linux" ]
tags = [ "ceph", "docker" ]
+++

# Ceph 是什么

Ceph 是一个开源的分布式存储系统，提供了对象存储、块存储和文件系统三种存储接口。Ceph 将数据存储在逻辑存储池中，使用 CRUSH 分布式算法决定如何将数据分散存储在集群的各个节点上，以实现高可用性和数据冗余。

## 特点

* 开源
* 部署简单
* 可靠性好
* 性能高
* 分布式，可扩展性强
* 客户端支持多语言

## 组件

搭建 Ceph 集群至少要包括一个 MON（Monitor）节点、一个MGR（Manager）节点和多个 OSD（Object Storage Daemon）节点，OSD 节点数量由我们要保存的数据副本数量决定，比如我们要将数据集存储三份，就需要部署至少三个 OSD 节点。

下面是各个节点介绍：

- OSD（Object Storage Daemon）: 集群中所有数据与对象的存储的守护进程。负责管理自盘上的数据块（数据存储、数据复制、数据恢复），起到复制/平衡/恢复数/指定数据读写操作等作用。如果要保证高可用，至少需要部署三个节点。

- MON（Monitor）: 监控集群状态，维护 Ceph 集群的状态信息、配置信息和映射信息，确保集群元数据的一致性，协调集群节点间数据的分布和恢复。维护 cluster MAP 表，保证集群数据一致性。如果要保证高可用，至少需要部署三个节点。

- MDS（Metadata Server）：元数据服务，保存文件系统服务的元数据（OBJ/Block不需要该服务）。负责管理文件系统的目录结构、文件和目录的元数据信息，为 CephFS（Ceph的分布式文件系统）提供元数据服务。块存储和对象存储不需要部署 MDS。

- MGR（Manager）：负责收集 Ceph 集群的状态信息（OSD、MON、MDS 的性能指标、健康状况等），并提供了可视化的仪表板（Ceph Dashboard）方便用户查看。如果要保证高可用，至少需要部署两个节点。

- RGW（Rados Gateway）：提供与Amazon S3和Swift兼容的RESTful API的gateway服务。提供了 RESTful API，允许用户发送 HTTP/HTTPS 请求访问和管理存储在 Ceph 集群中的数据，支持 Amazon S3 API 和 OpenStack Swift API。

## 环境准备

3台网络互通的 CentOS 7 主机：

|    主机名称       | 主机IP        | 角色          | 说明 |
| ----------      | ---           | ---          |  ---     |     
| chaos-1         |  172.20.30.1  |  ceph-admin  |  osd、mon、mds、mgr、rgw |
| chaos-2         |  172.20.30.2  |  ceph-1      |  osd、mon |
| chaos-3         |  172.20.30.3  |  ceph-2      |  osd、mon |



# 部署

**拉取镜像**

- [DockerHub 镜像地址](https://hub.docker.com/r/ceph/daemon/tags)
- x86_64 架构
```bash
docker pull ceph/daemon:master-4d96298-nautilus-centos-7-x86_64
docker image tag ceph/daemon:master-4d96298-nautilus-centos-7-x86_64 ceph/daemon:latest
```

**初始化ceph挂载目录(每个节点都执行)**
```bash
rm -rf /etc/ceph /var/lib/ceph /var/log/ceph
mkdir -p /etc/ceph /var/lib/ceph /var/log/ceph
```

**启动 Monitor 节点**

1、启动第一个 Monitor 节点(在 172.20.30.1 上执行)

```bash
docker run -d --net=host --restart always \
  -v /etc/ceph:/etc/ceph \
  -v /var/lib/ceph/:/var/lib/ceph/ \
  -v /var/log/ceph/:/var/log/ceph/ \
  -e MON_IP=172.20.30.1 \
  -e CEPH_PUBLIC_NETWORK=172.20.30.0/24 \
  --name="ceph-mon" \
  ceph/daemon:latest mon
```

2、然后将 `/etc/ceph` 目录拷到其他两个节点（172.20.30.2、172.20.30.3），使用 `scp` 命令远程拷贝:

```bash
scp -r /etc/ceph root@172.20.30.2:/etc/
scp -r /etc/ceph root@172.20.30.3:/etc/
```

3、启动第二个 Monitor 节点(在 172.20.30.2 上执行)

```bash
docker run -d --net=host --restart always \
  -v /etc/ceph:/etc/ceph \
  -v /var/lib/ceph/:/var/lib/ceph/ \
  -v /var/log/ceph/:/var/log/ceph/ \
  -e MON_IP=172.20.30.2 \
  -e CEPH_PUBLIC_NETWORK=172.20.30.0/24 \
  --name="ceph-mon" \
  ceph/daemon:latest mon
```

4、启动第三个 Monitor 节点(在 172.20.30.3 上执行)

```bash
docker run -d --net=host --restart always \
  -v /etc/ceph:/etc/ceph \
  -v /var/lib/ceph/:/var/lib/ceph/ \
  -v /var/log/ceph/:/var/log/ceph/ \
  -e MON_IP=172.20.30.3 \
  -e CEPH_PUBLIC_NETWORK=172.20.30.0/24 \
  --name="ceph-mon" \
  ceph/daemon:latest mon
```

5、查看 ceph 集群状态

```bash
[root@chaos-1 opt]# docker exec ceph-mon ceph -s
  cluster:
    id:     96711b46-de19-4cdc-b0ba-8b3e5a7960b9
    health: HEALTH_WARN
            mons are allowing insecure global_id reclaim

  services:
    mon: 3 daemons, quorum chaos-1,chaos-2,chaos-3 (age 71s)
    mgr: no daemons active
    osd: 0 osds: 0 up, 0 in

  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0 B
    usage:   0 B used, 0 B / 0 B avail
    pgs:

[root@chaos-1 opt]#
```

**启动mgr节点(每个节点执行同样的run命令)**

1、mgr模块用于分担monitor部分扩展功能，减轻monitor负担
```bash
docker run -d --net=host --restart=always \
  --privileged=true \
  --pid=host \
  --name="ceph-mgr" \
  -v /etc/ceph:/etc/ceph \
  -v /var/lib/ceph/:/var/lib/ceph/ \
  ceph/daemon:latest mgr
```

2、查看集群状态
```bash
[root@chaos-1 opt]# docker exec ceph-mon ceph -s
  cluster:
    id:     96711b46-de19-4cdc-b0ba-8b3e5a7960b9
    health: HEALTH_WARN
            OSD count 0 < osd_pool_default_size 3
            mons are allowing insecure global_id reclaim

  services:
    mon: 3 daemons, quorum chaos-1,chaos-2,chaos-3 (age 8m)
    mgr: chaos-1(active, since 2m), standbys: chaos-3, chaos-2
    osd: 0 osds: 0 up, 0 in

  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0 B
    usage:   0 B used, 0 B / 0 B avail
    pgs:

[root@chaos-1 opt]#
```

**启动 OSD 节点(每个节点执行)**

1、往每个 OSD 节点的 `/etc/ceph/ceph.conf` 文件追加下面内容
```bash
cat >> /etc/ceph/ceph.conf <<EOF
osd max object name len = 256
osd max object namespace len = 64
EOF
```

[Ceph osd启动报错osd init failed (36) File name too long](https://www.cnblogs.com/styshoo/p/6518698.html)

2、导出osd用于连ceph集群的 keyring
```bash
docker exec ceph-mon ceph auth get client.bootstrap-osd -o /var/lib/ceph/bootstrap-osd/ceph.keyring
```

3、创建osd的存储目录
```bash
mkdir -p /data/ceph/osd/vdb
```

4、启动osd
```bash
docker run -d --privileged=true --name=ceph-osdvdb --net=host \
  -v /etc/ceph:/etc/ceph \
  -v /var/lib/ceph/:/var/lib/ceph/ \
  -v /data/ceph/osd/vdb:/var/lib/ceph/osd \
  -e OSD_TYPE=directory \
  -v /etc/localtime:/etc/localtime:ro \
  ceph/daemon:latest osd
```

6、查看集群状态

```bash
[root@chaos-1 opt]# docker exec ceph-mon ceph -s
  cluster:
    id:     96711b46-de19-4cdc-b0ba-8b3e5a7960b9
    health: HEALTH_WARN
            mons are allowing insecure global_id reclaim

  services:
    mon: 3 daemons, quorum chaos-1,chaos-2,chaos-3 (age 25m)
    mgr: chaos-1(active, since 19m), standbys: chaos-3, chaos-2
    osd: 3 osds: 3 up (since 42s), 3 in (since 42s)

  task status:

  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0 B
    usage:   3.0 GiB used, 297 GiB / 300 GiB avail
    pgs:

[root@chaos-1 opt]#
```

**启动gateway节点**

1、导出rgw用于连接集群的keyring
```bash
docker exec ceph-mon ceph auth get client.bootstrap-rgw -o /var/lib/ceph/bootstrap-rgw/ceph.keyring
```

2、运行rgw节点, 可以启动一个或多个
```bash
docker run -d --net=host --privileged=true --name=ceph-rgw \
  -v /var/lib/ceph/:/var/lib/ceph/ \
  -v /etc/ceph:/etc/ceph \
  -v /etc/localtime:/etc/localtime:ro \
  -e RGW_NAME=rgw0 \
  ceph/daemon:latest rgw
```

3、查看集群状态
```bash
[root@chaos-1 opt]# docker exec ceph-mon ceph -s
  cluster:
    id:     96711b46-de19-4cdc-b0ba-8b3e5a7960b9
    health: HEALTH_WARN
            mons are allowing insecure global_id reclaim

  services:
    mon: 3 daemons, quorum chaos-1,chaos-2,chaos-3 (age 42m)
    mgr: chaos-1(active, since 35m), standbys: chaos-3, chaos-2
    osd: 3 osds: 3 up (since 17m), 3 in (since 17m)

  data:
    pools:   4 pools, 128 pgs
    objects: 9 objects, 1.2 KiB
    usage:   3.0 GiB used, 297 GiB / 300 GiB avail
    pgs:     41.406% pgs unknown
             75 active+clean
             53 unknown

  io:
    client:   767 B/s rd, 511 B/s wr, 0 op/s rd, 0 op/s wr

[root@chaos-1 opt]#
```

**启动dashboard可视化管理页面**

1、开启dashboard模块并禁用ssl(也可以用ssl, 需额外配置ssl证书)

```bash
docker exec ceph-rgw ceph mgr module enable dashboard
# 关闭https
docker exec ceph-rgw ceph config set mgr mgr/dashboard/ssl false
docker exec ceph-rgw ceph mgr module disable dashboard
docker exec ceph-rgw ceph mgr module enable dashboard
```

2、设置UI管理的host:port, 登录名及密码

我的虚拟主机也是容器环境，为了能在宿主机访问容器中部署的 ceph，我这里设置的IP为 `0.0.0.0`
```bash
docker exec ceph-rgw ceph config set mgr mgr/dashboard/server_addr 172.20.30.1
或者
docker exec ceph-rgw ceph config set mgr mgr/dashboard/server_addr 0.0.0.0
docker exec ceph-rgw ceph config set mgr mgr/dashboard/server_port 18080
```

验证是否配置成功：
```bash
docker exec ceph-rgw ceph mgr services
```

3、设置登录名及密码
```bash
docker exec ceph-rgw ceph dashboard ac-user-create <自定义user> <自定义pwd> administrator
docker exec ceph-rgw ceph dashboard ac-user-create ceph cephpass administrator
```

用户名：ceph
密码：cephpass

设置密码时我遇到下面错误：
```bash
[root@chaos-1 opt]# docker exec ceph-rgw ceph dashboard ac-user-create ceph cephpass administrator
Error EINVAL: Please specify the file containing the password/secret with "-i" option.
[root@chaos-1 opt]#
```

最新的ceph dashboard不支持直接在命令行里面创建用户的密码，需要先创建一个包含用户密码的文件。进入到 `ceph-rgw` 容器执行：
```bash
echo "cephpass" > ~/password.txt
```

退出容器重新执行：
```bash
[root@chaos-1 opt]# docker exec ceph-rgw ceph dashboard ac-user-create ceph -i ~/password.txt administrator
{"username": "ceph", "lastUpdate": 1730350784, "name": null, "roles": ["administrator"], "password": "$2b$12$5oxhctLdKYpz0DpT4WExheYu0tK/mghisrFawUEkoLUpgC0xeX9vy", "email": null}
[root@chaos-1 opt]#
```

4、登录并进入UI首页(http://<server_addr>:18080)

![alt text](/images/linux/ceph/10.png)

说明：我的虚拟主机也是容器环境，为了能在宿主机访问容器中部署的 ceph，容器的18080端口我映射的宿主机端口是20001。


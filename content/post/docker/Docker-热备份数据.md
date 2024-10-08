+++
title = '热备份数据'
date = 2018-11-18T00:16:03+08:00
draft = true
categories = [ "Docker" ]
tags = [ "docker" ]
+++

# 数据备份分类

## 冷备份

- 冷备份是关闭数据库时候的备份方式，通常做法是拷贝数据文件
- 冷备份是最简单最安全的一种备份方式
- 大型网站无法做到关闭业务备份数据，所以冷备份不是最佳选择

![](/images/docker/226.png)

其中比如 `msyqldump` 备份是冷备份的一种，另外还有一种方法是拷贝数据文件，将来在还原的时候把数据文件覆盖到MySQL上，这就实现了还原。关于冷备份最大的问题就是不能在系统运营中去备份，必须要让系统停机去备份数据。

如果在数据库集群中偏要使用冷备份，就是在集群中让某一个节点下线，把这个节点下线之后再去备份这个节点的数据，这个就可以实现冷备份，而其他在上线的节点是正常可以开展业务的，然后下线的节点数据备份成功之后再让它上线，跟其他的节点一同步，那么下线的节点数据和其他上线的节点的数据就保持实时的更新了。

## 热备份

- 热备份是在系统运行的状态下备份数据，也是难度最大的备份
- MySQL常见的热备份有LVM和XtraBackup两种方案
- 建议使用XtraBackup热备份MySQL

其中 `LVM` 方式是Linux自带技术，Linux对某一个分区创建快照实现对分区数据的备份，原则上LVM可以备份的数据有很多，例如MySQL、Oracle、MongoDB都可以，因为它们备份的是分区里面的数据，无论什么数据它们的数据都保存到Linux分区上，所以LVM原则上是可以备份任何数据库的，但是这种方式存在缺点，就是在备份的时候需要对数据库加锁，数据库只能读取数据，不能写入数据，这种情况对业务的开展还是有一定影响的，而且在操作LVM分区的时候指令较多，所以这种备份方式比较麻烦。

`XtraBackup` 备份方式有很多优点，比如它不需要锁表就可以备份数据，写入数据和备份数据可以同时进行，另外它还是免费开源的备份数据方案。

# XtraBackup热备份

## 优势

- XtraBackup备份过程不锁表、快速可靠
- XtraBackup备份过程不会打断正在执行的事务，也就是说在备份过程中同时执行增删改查的操作依然是可以的
- XtraBackup能够基于压缩等功能节约磁盘空间和流量

## 全量备份和增量备份

XtraBackup热备份分为全量备份和增量备份

- 全量备份是备份全部数据。因为备份的数据比较多，所以备份的速度比较慢，备份过程时间长，对硬盘占用空间大

![](/images/docker/98.png)

对数据库的第一次备份可以采用全量备份，后续的备份就可以采用增量备份，这样备份速度比较快，另外减小硬盘压力。

- 增量备份是只备份变化的那部分数据。备份时间短，占用空间小

![](/images/docker/99.png)

## 准备

XtraBackup热备份是安装在数据库所在的节点容器内，它备份出来的数据会直接保存在容器中，这样很不好，我们应该将备份出来的数据导出到宿主机上，应该采用映射的技术，所以需要在宿主机上创建数据卷，然后将数据卷映射到某一个数据库的节点上，然后再通过热备工具备份出来的数据在宿主机上就可以看到了。

* 创建数据卷

```
docker volume create backup
```

* 创建完之后选择一个数据库节点作为映射，这里选择node1节点，首先要将数据库节点停止

```
docker stop node1
```

当初创建这个node1节点的时候创建出来的容器在使用创建参数并没有映射backup数据卷，而是映射的v1的数卷，所以说现在启动的时候没有办法往里面添加参数了，所以现在把node1节点停止主要是将其删除，然后再创建一个新的node1节点，创建的指令里映射backup数据卷。

但是现在创建新的node1容器，那原来的node1里面的数据就删掉了吗，这个原来node1容器创建的时候就已经映射了v1的数据卷保存容器内数据库的数据，所以把node1容器删掉再创建一个新的node1是没有影响的，只要把v1的数据卷映射到新的node1容器中，那么启动的数据库就还有之前的数据了。

* 删除node1容器

```
docker rm node1
```

* 创建新的node1

```
docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=abc123456 -e CLUSTER_NAME=PXC -e XTRABACKUP_PASSWORD=abc123456 -v v1:/var/lib/mysql -v backup:/data --privileged -e CLUSTER_JOIN=node2 --name=node1 --net=net1 --ip 172.18.0.2 pxc
```

`-v backup:/data`: 表示把宿主机 `backup` 数据卷映射到容器中的 `/data` 目录
`-e CLUSTER_JOIN=node2` 表示现在创建的 `node1` 需要和哪一个节点进行同步，原来的数据库节点创建出来之后其他的节点跟 `node1` 同步的，现在 `node1` 停止掉之后创建一个新的 `node1` 要与现在的数据库集群里的某一个节点进行同步，这里选择的 `node2` 节点

* 如果在创建的过程中出现了下面的错误

![](/images/docker/100.png)

这是因为之间系统将防火墙关闭的原因，在创建之前仍需将node节点先停止再删除，解决方法

```
docker stop node1
docker rm node1
systemctl start firewalld
docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=abc123456 -e CLUSTER_NAME=PXC -e XTRABACKUP_PASSWORD=abc123456 -v v1:/var/lib/mysql -v backup:/data --privileged -e CLUSTER_JOIN=node2 --name=node1 --net=net1 --ip 172.18.0.2 pxc
```

然后再去重新创建node1节点

![](/images/docker/101.png)

## PXC全量备份步骤

- PXC容器中安装XtraBackup,并执行备份

### 指令

```
apt-get update
apt-get install percona-xtrabackup-24
innobackupex --user=root --password=abc123456 /data/back/full
```

### 实操

1. 进入node1容器

```
docker exec -it node1 bash
```

2. 更新

```
apt-get update
```

如果这个时候会出现下面没有权限的提示

解决方法: 退出容器以下面指令重新进入容器,以root用户登陆docker内linux

```
docker exec -it -u 0 node1 bash
```

![](/images/docker/102.png)

3. 安装热备工具

```
apt-get install percona-xtrabackup-24
```

4. 粘贴全量备份语句，然后它就可以立即执行备份了

```
innobackupex --user=root --password=abc123456 /data/backup/full
```

备份完之后提示完成

![](/images/docker/103.png)

5. 进入备份目录

```
cd /data/back/full
```

![](/images/docker/104.png)

6. 这是容器内的数据，现在退出查看映射到宿主机上也就是容器外的备份数据

```
exit
docker inspect back
```

![](/images/docker/105.png)

到这里全量备份已经执行成功了，这是热备份，我们可以一边写入数据一边备份数据

## PXC全量恢复步骤

使用了backup全量热备份数据备份出来的数据将来可能会用得上，在数据库出现故障的时候可以利用这个备份去还原数据库，数据库的备份分为冷备份和热备份，但是数据库的还原只有一种就是冷还原，假如有热还原一边开展业务数据，一边还原数据，相互覆盖，还原出来的数据肯定是有问题的，结果难以想象。

- 数据库可以热备份，但是不能热还原。为了避免恢复过程中的数据同步，我们采用的空白的MySQL还原数据，然后再建立PXC集群

PXC集群中某个节点直接使用冷还原会造成PXC集群的故障，因为其他的节点不知道怎么和执行冷还原的节点进行同步数据，真正的PXC集群实现冷还原的步骤是需要把PXC集群先解散，然后将数据节点删掉，然后再创建一个新的数据节点，这个数据节点执行冷还原，冷还原之后再去创建其他的节点，和执行还原的节点进行数据同步，这样PXC集群就重新搭建起来了

- 还原数据前要将当初热备份未提交的事务回滚(删除)，然后执行还原，还原数据之后重新启动MySQL

具体还原的流程需要把当初备份的时候，因为数据是热备份的，所以当初在热备份的时候有些事务没有提交事务就被备份起来了，所以在还原的时候首先是要把那些没有提交的事务删除掉，然后再执行还原，还原之后重启MySQL。

### 指令

```
#删除MySQL内部数据
rm -rf /var/lib/mysql/*
#将没有提交的数据进行回滚 后面是热备的数据目录
innobackupex --user=root --password=abc123456 --apply-back /data/back/full/2018-11-18_05-09-07/
#还原数据
innobackupex --user=root --password=abc123456 --copy-back /data/backup/full/2018-11018_05-09-07/
```

### 实操

在还原之前将数据库插入一条记录，此时数据库中有4条记录

![](/images/docker/106.png)

此时备份的数据和现有的数据是不一样的，备份是的有三条记录，现在有四条记录，在经过还原之后应该有三条记录

1. 将5个pxc节点停止

```
docker stop node1 node2 node3 node4 node5
```

2. 将5个pxc节点删除

```
docker rm node1 node2 node3 node4 node5
```

3. 删除映射的数据卷

```
docker volume rm v1 v2 v3 v4 v5
```

4. 创建新的数据卷v1

```
docker volume create v1
```

5. 打开之前关掉的防火墙（可选）,我没有执行


```
systemctl start firewalld
```

6. 然后启动一个pxc节点node1

```
docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=abc123456 -e CLUSTER_NAME=PXC -e XTRABACKUP_PASSWORD=abc123456 -v v1:/var/lib/mysql -v backup:/data --privileged --name=node1 --net=net1 --ip 172.18.0.2 pxc

docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=abc123456 -e CLUSTER_NAME=PXC -e XTRABACKUP_PASSWORD=abc123456 -v v1:/var/lib/mysql --privileged --name=node1 --net=net1 --ip 172.18.0.2 pxc
```

![](/images/docker/107.png)

7. 然后进入容器

```
docker exec -it -u 0 node1 bash
```

或者 

```
docker exec -it node1 bash
```

8. 清空MySQL数据指令

```
rm -rf /var/lib/mysql/*
```

9. 回滚没有提交的指令

```
innobackupex --user=root --password=abc123456 --apply-back /data/backup/full/2018-11-18_05-49-46/
```

![](/images/docker/108.png)

10. 执行冷还原

```
innobackupex --user=root --password=abc123456 --copy-back /data/backup/full/2018-11-18_05-49-46/
```

还原成功

![](/images/docker/109.png)

11. 退出容器，启动node1节点

```
exit
docker stop node1 
docker start node1
```

![](/images/docker/110.png)

12. 查看node1节点里面的数据

![](/images/docker/111.png)
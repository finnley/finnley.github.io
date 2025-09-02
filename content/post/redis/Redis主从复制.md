+++
title = 'Redis主从复制'
date = 2021-03-02T12:19:40+08:00
draft = false
categories = [ "Redis" ]
tags = [ "redis", "DBA" ]
+++

## 单机Redis存在的问题

什么是单机？就是在一台机器上部署一个Redis节点。

- 机器故障 

一旦该节点出现故障，则可能无法在短时间内恢复，客户端则无法连接Redis，严重会影响生产。这就是单机存在的问题。

- 容量瓶颈
- QPS瓶颈

## 主从复制的基本作用

一主一从、一主多从

- 备份。为一份数据提供多个副本。
- 读写分离。扩展读写性能。

## 主从设置

### 命令方式



### 配置方式

### 操作

**master节点**

- IP：127.0.0.1
- PORT：6379
- 配置：

```bash
cp redis.conf config/
cd config
cp redis.conf redis-6379.conf
vim redis-6379.conf
```

主要配置修改如下，其他配置默认：
```bash
# cat redis-6379.conf | grep -v "#" | grep -v "^$"
# 设置后台守护进程启动
daemonize yes
# 设置启动文件
pidfile /var/run/redis-6379.pid
# 配置端口
port 6379
# 配置日志文件
logfile "6379.log"
# 注释掉下面自动生成RDB配置
#save 900 1
#save 300 10
#save 60 10000
# 配置dbfilename
dbfilename dump-6379.rdb
# 设置数据目录
dir /opt/redis/data
```

**slave节点**

- IP：127.0.0.1
- PORT：6380
- 配置：

```bash
cp redis.conf config/
cd config
cp redis-6379.conf redis-6380.conf
vim redis-6380.conf
```

主要配置修改如下，其他配置默认：
```bash
# cat redis-6380.conf | grep -v "#" | grep -v "^$"
# 设置后台守护进程启动
daemonize yes
# 设置启动文件
pidfile /var/run/redis-6380.pid
# 配置端口
port 6380
# 配置日志文件
logfile "6380.log"
# 注释掉下面自动生成RDB配置
#save 900 1
#save 300 10
#save 60 10000
# 配置dbfilename
dbfilename dump-6380.rdb
# 设置数据目录
dir /opt/redis/data
# 配置slave复制的master节点IP和Port
slaveof 127.0.0.1 6379
```

**启动**

首先启动6379maste节点：
```bash
# cd /opt/redis/config
# redis-server redis-6379.conf
# ps -ef | grep redis- | grep -v grep
root     2393156       1 18 09:39 ?        00:00:09 redis-server *:6379
# 查看分片
# redis-cli
127.0.0.1:6379> info replication
# Replication
# 可以看到它是master角色的状态，连接的从节点目前是0个
role:master
connected_slaves:0
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
127.0.0.1:6379>
```

启动从节点：
```bash
# redis-server redis-6380.conf
# ps -ef | grep redis- | grep -v grep
root     2393156       1  5 09:39 ?        00:00:11 redis-server *:6379
root     2393166       1 70 09:43 ?        00:00:08 redis-server *:6380
# 查看slave节点分片信息
# redis-cli -p 6380 info replication
# Replication
# 可以看到它是slave角色的状态，连接的主节点IP和Port是127.0.0.1和6379，以及和主节点的连接状态是up
role:slave
master_host:127.0.0.1
master_port:6379
master_link_status:up
master_last_io_seconds_ago:7
master_sync_in_progress:0
slave_repl_offset:99
slave_priority:100
slave_read_only:1
connected_slaves:0
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
```

**验证**

在主节点上设置一条数据，然后观察再从节点去查询这条数据：
```bash
# redis-cli -h 127.0.0.1 -p 6379
127.0.0.1:6379> set hello world
OK
127.0.0.1:6379> exit
# redis-cli -h 127.0.0.1 -p 6380
127.0.0.1:6380> get hello
"world"
# 从节点无法执行写入操作，只能读取
127.0.0.1:6380> set hello redis
(error) READONLY You can't write against a read only slave.
127.0.0.1:6380>
```

![](/images/redis/110.png)

**查看日志**

master:
```bash
...
2393156:M 13 Nov 09:40:07.157 * DB loaded from disk: 8.606 seconds
2393156:M 13 Nov 09:40:07.157 * The server is now ready to accept connections on port 6379
# Slave 127.0.0.1:6380 要想做同步操作，也就是复制
2393156:M 13 Nov 09:43:08.169 * Slave 127.0.0.1:6380 asks for synchronization
# 请求为全量同步请求，也就是全量复制
2393156:M 13 Nov 09:43:08.169 * Full resync requested by slave 127.0.0.1:6380
2393156:M 13 Nov 09:43:08.169 * Starting BGSAVE for SYNC with target: disk
# 接着master主节点会做一个bgsave操作，实际上就是生成一个RDB，将快照通过网络做同步
2393156:M 13 Nov 09:43:08.174 * Background saving started by pid 2393169
2393169:C 13 Nov 09:43:11.783 * DB saved on disk
2393169:C 13 Nov 09:43:11.785 * RDB: 0 MB of memory used by copy-on-write
2393156:M 13 Nov 09:43:11.883 * Background saving terminated with success
2393156:M 13 Nov 09:43:12.139 * Synchronization with slave 127.0.0.1:6380 succeeded
```

slave:
```bash
...
2393166:S 13 Nov 09:43:08.168 * The server is now ready to accept connections on port 6380
# slave节点连接6379
2393166:S 13 Nov 09:43:08.168 * Connecting to MASTER 127.0.0.1:6379
# 主从复制开始
2393166:S 13 Nov 09:43:08.168 * MASTER <-> SLAVE sync started
2393166:S 13 Nov 09:43:08.168 * Non blocking connect for SYNC fired the event.
2393166:S 13 Nov 09:43:08.168 * Master replied to PING, replication can continue...
2393166:S 13 Nov 09:43:08.169 * Partial resynchronization not possible (no cached master)
# 做复制操作从节点就需要主节点的run id，可以通过执行 redis-cli -p 6379 info 查看master节点的run id
2393166:S 13 Nov 09:43:08.174 * Full resync from master: cb7f26f8a4b70e4c91524bd24486c69833f01018:1
# 从master节点多少字节，接收的就是rdb文件
2393166:S 13 Nov 09:43:11.884 * MASTER <-> SLAVE sync: receiving 247777833 bytes from master
# 只要是从节点去复制主节点，都会先做数据的清除
2393166:S 13 Nov 09:43:12.146 * MASTER <-> SLAVE sync: Flushing old data
# 加载rdb文件到内存，完成复制
2393166:S 13 Nov 09:43:12.146 * MASTER <-> SLAVE sync: Loading DB in memory
2393166:S 13 Nov 09:43:20.683 * MASTER <-> SLAVE sync: Finished with success
```

**移除主从关系**

连接6380从节点，执行下面命令：
```bash
[root@vm-6 data]# redis-cli -p 6380
127.0.0.1:6380> slaveof no one
OK
127.0.0.1:6380> info replication
# Replication
# 此时已取消主从关系变成了master
role:master
connected_slaves:0
master_repl_offset:1305
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
127.0.0.1:6380> exit
# 连接 6379 查看主从关系，已经没有了从节点
[root@vm-6 data]# redis-cli -p 6379
127.0.0.1:6379> info replication
# Replication
role:master
connected_slaves:0
master_repl_offset:1487
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:2
repl_backlog_histlen:1486
127.0.0.1:6379>
# 设置测试数据
127.0.0.1:6379> mset a b c d e f g h
OK
127.0.0.1:6379> dbsize
(integer) 10000005
127.0.0.1:6379> exit
[root@vm-6 data]# redis-cli -p 6380
127.0.0.1:6380> dbsize
(integer) 10000005
127.0.0.1:6380> flushdb
OK
(6.10s)
127.0.0.1:6380> set hello 6380
OK
127.0.0.1:6380>
# 执行建立主从复制关系后查看是否还存在上面设置的key
127.0.0.1:6380> slaveof 127.0.0.1 6379
OK
127.0.0.1:6380> get hello
(nil)
```

### 全量复制与部分复制

**run_id**

redis 每次启用的时候都会有一个随机的ID来作为redis的标识，重启之后又会重新生成。
```bash
# redis-cli -p 6379 info server | grep run
run_id:b19c87443c8cf5bb4797800cd81fd4e080e16d00
[root@vm-6 config]# redis-cli -p 6380 info server | grep run
run_id:d8afb2262ce1b69876df9854bcf365614c2afde4
```

**偏移量**

```bash
# redis-cli -p 6379 info replication
# Replication
role:master
connected_slaves:1
slave0:ip=127.0.0.1,port=6380,state=online,offset=406,lag=1
# 这就是偏移量
master_repl_offset:406
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:2
repl_backlog_histlen:405

# redis-cli -p 6379 set hello redis
OK
# redis-cli -p 6379 info replication
# Replication
role:master
connected_slaves:1
slave0:ip=127.0.0.1,port=6380,state=online,offset=469,lag=0
# 更新了一条数据后的偏移量
master_repl_offset:469
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:2
repl_backlog_histlen:468

# redis-cli -p 6380 info replication
# Replication
role:slave
master_host:127.0.0.1
master_port:6379
master_link_status:up
master_last_io_seconds_ago:7
master_sync_in_progress:0
# 查看从节点的偏移量
slave_repl_offset:553
slave_priority:100
slave_read_only:1
connected_slaves:0
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0

# redis-cli -p 6379 info replication
# Replication
role:master
connected_slaves:1
slave0:ip=127.0.0.1,port=6380,state=online,offset=609,lag=0
# 再查看主节点的偏移量已经发生了变化，实际上主从之间偏移量是个同步更新的状态，它们之间也会有一些操作，
# 从节点会向主节点做上报，会将从节点的状态同步给主节点，这样主节点就会知道从节点的偏移量
master_repl_offset:609
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:2
repl_backlog_histlen:608
```

**全量复制开销**

- bgsave的时间
- RDB文件在网络传输的时间
- 从节点清空数据的时间
- 从节点加载RDB的时间
- 可能的AOF时间

部分复制



## 小结

- 一个master可以有多个slave，一个slave 也可以有它自己的slave，那这个slave也就充当了master的角色
- 一个slave只能有一个master
- 数据流向为单向，master到slave
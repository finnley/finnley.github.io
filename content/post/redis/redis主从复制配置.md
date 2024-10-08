+++
title = 'Redis主从复制配置'
date = 2024-06-10T11:31:09+08:00
draft = true
categories = [ "Redis" ]
tags = [ "redis", "DBA" ]
+++

## 主从复制配置

配置入下：
master 端口：6379
slave 端口：6380

### 主节点配置

将标准配置拷贝到 config 目录下：
```bash
# pwd
/opt/soft/redis
# cp redis.conf config/redis-6379.conf
#
```

redis-6379.conf 配置如下：

1、以守护进程方式启动 <br>
2、修改进程ID <br>
3、端口：6379 <br>
4、日志文件：6379.log <br>
5、注释掉save 自动rdb生成配置 <br>
6、设置rdb文件 <br>
7、工作目录 <br>


```bash
daemonize yes
pidfile /var/run/redis-6379.pid
port 6379
logfile "6379.log"
#save 900 1
#save 300 10
#save 60 10000
dbfilename dump-6379.rdb
dir /opt/soft/redis/data
```

### 从节点配置

直接拷贝一份redis-6379.conf 文件到 config 目录下：
```bash
# pwd
/opt/soft/redis/config
# cp redis-6379.conf redis-6380.conf
#
```

6380 节点会去复制 6379 节点的数据：
1、以守护进程方式启动 <br>
2、修改进程ID <br>
3、端口：6380 <br>
3、端口：6380.log <br>
5、注释掉save 自动rdb生成配置 <br>
6、设置rdb文件 <br>
7、工作目录 <br>
8、复制主节点数据，由于当前都是安装本机上，所有这里slaveof设置的master ip是127.0.0.1


redis-6380.conf 配置如下：
```bash
daemonize yes
pidfile /var/run/redis-6380.pid
port 6380
logfile "6380.log"
#save 900 1
#save 300 10
#save 60 10000
dbfilename dump-6380.rdb
dir /opt/soft/redis/data
slaveof 127.0.0.1 6379
```

### 启动

首先启动 6379 节点：
```bash
redis-server redis-6379.conf
```

连接：
查看分片，有一个分片叫 replication，会看到分片是 master 状态：
```bash
# redis-cli
127.0.0.1:6379> info replication
# Replication
role:master
connected_slaves:0
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
127.0.0.1:6379>
```

退出并启动一个从节点：
```bash
redis-server redis-6380.conf
```

查询信息：
```bash
# redis-cli -p 6380 info replication
# Replication
role:slave                      # 角色是从
master_host:127.0.0.1           # 由于是从节点，会标明主节点的IP以及端口
master_port:6379
master_link_status:up           # 当前和主节点的连接状态，up表示正常连接状态
master_last_io_seconds_ago:4
master_sync_in_progress:0
slave_repl_offset:85
slave_priority:100
slave_read_only:1
connected_slaves:0
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
```

### 验证

首先登录到主节点执行set命令，接着退出主节点登录到从节点执行一下命令get命令：

```bash
# redis-cli
127.0.0.1:6379> set hello world
OK
127.0.0.1:6379> exit
[root@localhost config]# redis-cli -p 6380
127.0.0.1:6380> get hello
"world"
127.0.0.1:6380>
```

发现数据很快就在同步到了从节点。

如果在从节点上执行set操作，会看到如下提示：

```bash
127.0.0.1:6380> set hello redis
(error) READONLY You can't write against a read only slave.
```

也就是说从节点只能做读操作。

###  日志

查看主节点日志：

```bash
# cat 6379.log
2046:M 10 Jun 12:45:59.823 * Increased maximum number of open files to 10032 (it was originally set to 1024).
                _._
           _.-``__ ''-._
      _.-``    `.  `_.  ''-._           Redis 3.0.7 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 2046
  `-._    `-._  `-./  _.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |           http://redis.io
  `-._    `-._`-.__.-'_.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |
  `-._    `-._`-.__.-'_.-'    _.-'
      `-._    `-.__.-'    _.-'
          `-._        _.-'
              `-.__.-'

2046:M 10 Jun 12:45:59.835 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
2046:M 10 Jun 12:45:59.836 # Server started, Redis version 3.0.7
2046:M 10 Jun 12:45:59.836 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
2046:M 10 Jun 12:45:59.837 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
2046:M 10 Jun 12:45:59.838 * The server is now ready to accept connections on port 6379
2046:M 10 Jun 12:48:59.679 * Slave 127.0.0.1:6380 asks for synchronization
2046:M 10 Jun 12:48:59.680 * Full resync requested by slave 127.0.0.1:6380
2046:M 10 Jun 12:48:59.681 * Starting BGSAVE for SYNC with target: disk
2046:M 10 Jun 12:48:59.687 * Background saving started by pid 2061
2061:C 10 Jun 12:48:59.706 * DB saved on disk
2061:C 10 Jun 12:48:59.710 * RDB: 4 MB of memory used by copy-on-write
2046:M 10 Jun 12:48:59.800 * Background saving terminated with success
2046:M 10 Jun 12:48:59.804 * Synchronization with slave 127.0.0.1:6380 succeeded
```

注意日志中除了启动日子外的这样几句日志：

```bash
# 希望做同步操作，也就说希望做复制操作
2046:M 10 Jun 12:48:59.679 * Slave 127.0.0.1:6380 asks for synchronization
# 做全量同步请求，也就是全量复制
2046:M 10 Jun 12:48:59.680 * Full resync requested by slave 127.0.0.1:6380
# 做的bgsave操作，也就是将当前快照RDB做了一次同步
# 也就是RDB关闭还是自动配置关闭外，还是有场景如复制场景还是会做RDB的操作
2046:M 10 Jun 12:48:59.681 * Starting BGSAVE for SYNC with target: disk
2046:M 10 Jun 12:48:59.687 * Background saving started by pid 2061
# 持久化到磁盘
2046:M 10 Jun 12:48:59.687 * Background saving started by pid 2061
2061:C 10 Jun 12:48:59.706 * DB saved on disk
# 返回结果同步成功
2046:M 10 Jun 12:48:59.800 * Background saving terminated with success
2046:M 10 Jun 12:48:59.804 * Synchronization with slave 127.0.0.1:6380 succeeded
```

```bash
# cat 6380.log
2058:S 10 Jun 12:48:58.638 * Increased maximum number of open files to 10032 (it was originally set to 1024).
                _._
           _.-``__ ''-._
      _.-``    `.  `_.  ''-._           Redis 3.0.7 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6380
 |    `-._   `._    /     _.-'    |     PID: 2058
  `-._    `-._  `-./  _.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |           http://redis.io
  `-._    `-._`-.__.-'_.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |
  `-._    `-._`-.__.-'_.-'    _.-'
      `-._    `-.__.-'    _.-'
          `-._        _.-'
              `-.__.-'

2058:S 10 Jun 12:48:58.647 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
2058:S 10 Jun 12:48:58.648 # Server started, Redis version 3.0.7
2058:S 10 Jun 12:48:58.648 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
2058:S 10 Jun 12:48:58.648 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
2058:S 10 Jun 12:48:58.649 * The server is now ready to accept connections on port 6380
2058:S 10 Jun 12:48:59.652 * Connecting to MASTER 127.0.0.1:6379
2058:S 10 Jun 12:48:59.667 * MASTER <-> SLAVE sync started
2058:S 10 Jun 12:48:59.669 * Non blocking connect for SYNC fired the event.
2058:S 10 Jun 12:48:59.673 * Master replied to PING, replication can continue...
2058:S 10 Jun 12:48:59.678 * Partial resynchronization not possible (no cached master)
2058:S 10 Jun 12:48:59.692 * Full resync from master: 6b43d5141a6180e8fbea72e519fd2c239d4be614:1
2058:S 10 Jun 12:48:59.802 * MASTER <-> SLAVE sync: receiving 18 bytes from master
2058:S 10 Jun 12:48:59.805 * MASTER <-> SLAVE sync: Flushing old data
2058:S 10 Jun 12:48:59.806 * MASTER <-> SLAVE sync: Loading DB in memory
2058:S 10 Jun 12:48:59.807 * MASTER <-> SLAVE sync: Finished with success
```


注意下面的几句日志：
```bash
# 连接的主节点6379
2058:S 10 Jun 12:48:59.652 * Connecting to MASTER 127.0.0.1:6379
# 主从复制开始
2058:S 10 Jun 12:48:59.667 * MASTER <-> SLAVE sync started
# 类似于PING的操作
2058:S 10 Jun 12:48:59.669 * Non blocking connect for SYNC fired the event.
2058:S 10 Jun 12:48:59.673 * Master replied to PING, replication can continue...
# 做全量复制
2058:S 10 Jun 12:48:59.678 * Partial resynchronization not possible (no cached master)
# 拿到的 master的 run_id，每个redis启动的时候都有自己的 run_id，既然是要做复制就要知道主的run_id是什么
2058:S 10 Jun 12:48:59.692 * Full resync from master: 6b43d5141a6180e8fbea72e519fd2c239d4be614:1
# 从master，即rdb文件接收18字节数据
# 只要是从节点去复制主节点，就会将数据进行清除
2058:S 10 Jun 12:48:59.802 * MASTER <-> SLAVE sync: receiving 18 bytes from master
2058:S 10 Jun 12:48:59.805 * MASTER <-> SLAVE sync: Flushing old data
# 加载rdb 文件
2058:S 10 Jun 12:48:59.806 * MASTER <-> SLAVE sync: Loading DB in memory
# 同步完成，完成复制
2058:S 10 Jun 12:48:59.807 * MASTER <-> SLAVE sync: Finished with success
```

查看下主的rid:
```bash
# redis-cli info
# Server
...
run_id:6b43d5141a6180e8fbea72e519fd2c239d4be614
...
```

如果说master一开始是有数据的情况下，在做全量复制的时候会把原来的数据也全部拉过来，比如现在连接6380节点，执行下面命令：
```bash
# redis-cli -p 6380
127.0.0.1:6380> slaveof no one
OK
127.0.0.1:6380> info replication
# Replication
role:master
connected_slaves:0
master_repl_offset:2677
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
127.0.0.1:6380>
```

就可以看到它是一个master角色。接着到 6379 节点执行下面命令：
```bash
127.0.0.1:6380> exit
[root@localhost data]# redis-cli
127.0.0.1:6379> info replication
# Replication
role:master
connected_slaves:0
master_repl_offset:2803
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:2
repl_backlog_histlen:2802
127.0.0.1:6379>
```

发现6379也是master节点，没有salve节点，此时往6379节点写一些数据：
```bash
127.0.0.1:6379> mset a b c d e f g h
OK
127.0.0.1:6379> dbsize
(integer) 5
127.0.0.1:6379>
```

然后退出6379节点进入6380节点：
```bash
127.0.0.1:6379> exit
[root@localhost data]# redis-cli -p 6380
127.0.0.1:6380> dbsize
(integer) 1
127.0.0.1:6380>
```

发现此时两个节点数据是不同步的，现在在6380节点上数据清除，并设置一些不一样的数据：
```bash
127.0.0.1:6380>  flushall
OK
127.0.0.1:6380> set abc7380 hello
OK
127.0.0.1:6380> slaveof 127.0.0.1 6379
OK
127.0.0.1:6380> get abc6380
(nil)
127.0.0.1:6380> dbsize
(integer) 5
127.0.0.1:6380>
```

发现获取不到，现在是个新的从节点，数据也从master节点同步过来了。

## 全量复制和部分复制

redis 每次启动时都有一个随机的id来保证redis的唯一标识，但是它重启之后run_id就没了。

查看下 6379、6380 的run_id:
```bash
# redis-cli -p 6379 info server | grep run
run_id:6b43d5141a6180e8fbea72e519fd2c239d4be614
# redis-cli -p 6380 info server | grep run
run_id:60c187fbb8236dc653a7737d8b334a6650f69a2e
```

如果6380复制的6379，就会将6379 run_id 在6380节点上做个标识，但如果突然间发现6379 run_id 发生了变化，比如重启操作，就需要他的数据全部同步过来，这就是全量复制。 

偏移量就是数据量写入了多少个字节，可以记录写了多少数据，比如6379节点写入了数据，它会将这个偏移量同步到6380，当两个节点偏移量一致的时候就表示数据时完全同步的，如果6379的偏移量比6380的大，就表示主从不一致，主节点数据没有同步到从，在主节点上执行 `info replication` 命令这种偏移量 `master_repl_offset:2803`，下面做个实验：

```bash
# redis-cli -p 6379 set java hello
OK
[root@localhost data]# redis-cli -p 6379 info replication
# Replication
role:master
connected_slaves:1
slave0:ip=127.0.0.1,port=6380,state=online,offset=5072,lag=1
master_repl_offset:5072
...
```

接着看下从节点的偏移量：
```bash
# redis-cli -p 6380 info replication
# Replication
role:slave
master_host:127.0.0.1
master_port:6379
master_link_status:up
master_last_io_seconds_ago:4
master_sync_in_progress:0
slave_repl_offset:5142
slave_priority:100
slave_read_only:1
connected_slaves:0
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
```

在看下主节点的偏移量：
```bash
# redis-cli -p 6379 info replication
# Replication
role:master
connected_slaves:1
slave0:ip=127.0.0.1,port=6380,state=online,offset=5212,lag=1
master_repl_offset:5212
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:2
repl_backlog_histlen:5211
```

发现主节点的偏移量发生了变化，实际上他们的偏移量是个同步更新的状态，它们之间其实也会做一些操作。

另外从节点也会向主节点做一个上报操作，会将从节点的状态同步给主节点，这样主节点就可以知道从节点的偏移量，比如下面的6379主节点上知道slave的offset=5212，master_repl_offset:5212。
```bash
slave0:ip=127.0.0.1,port=6380,state=online,offset=5212,lag=1
master_repl_offset:5212
```

对于存在很多数据的master节点，此时slave节点去做复制，slave会将master之前的数据同步过来，同时还希望在同步的期间往master节点写入的数据也同步过来，一次来达到数据完全同步的效果。

redis 提供的全量复制的功能就能满足上面要求。

首先master会将当前的rdb文件同步给slave，在此同步期间往master写入的数据会单独文件记录，当rdb文件加载完之后，会通过偏移量的对比，将这期间写入的数据同步给slave。过程如下：

1、psync ?-1，做同步命令，可以实现全量复制和部分复制的功能，它有两个参数，一个参数是 run_id，一个参数是偏移量，也就是会向主节点报告我（从节点）的偏移量是多少，然后主节点会将偏移量之后的数据给从节点。

第一次同步，从节点并不知道主节点的run_id和偏移量是多少，就传一个？和-1

2、master 接收到同步命令之后，可以感知到是做全量复制，因为？表示从节点不知道主节点的run_id是多少，-1 表示 从节点不知道主节点的偏移量是多少，所以此时主节点会将自己的 run_id和偏移量告诉从节点

3、slave 接着回保存master的基本信息

4、之后master节点会做持久化相关操作的bgsave

主节点将全量数据和部分数据（从rdb开始生成到rdb传输这一过程中新产生的命令）进行保存，然后传输过去，
redis中有个复制缓冲区，它可以记录新写入的命令，这样就可以实现上面的效果

5、在send RDB之后

6、会做send buffer

7、之后从节点会清除老的数据

8、加载RDB文件，同时会将buffer数据加载进去

以上过程就保证了主从节点数据的同步，从而实现数据副本的功能。

![](/img/redis/40.png)





    
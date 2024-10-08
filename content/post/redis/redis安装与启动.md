+++
title = 'Redis安装与启动'
date = 2024-06-10T10:46:13+08:00
draft = true
categories = [ "Redis" ]
tags = [ "redis", "DBA" ]
+++

## redis安装启动

参考：[https://www.cnblogs.com/operationhome/p/10342258.html](https://www.cnblogs.com/operationhome/p/10342258.html)

### 下载

版本：redis 3.0.7

当前所在目录： 
```bash
$ pwd
/opt/soft
```

下载:
```bash
$ sudo wget https://download.redis.io/releases/redis-3.0.7.tar.gz
```

解压:
```bash
$ sudo tar -xvf redis-3.0.7.tar.gz
```

为了方便管理和未来的升级，通常在当前目录下创建软链:
```bash
$ sudo ln -s redis-3.0.7 redis
```

进入 redis 目录，编译安装，会在 `/usr/local/bin` 目录下生成可执行文件，之后就可以在任意目录下执行 redis 的可执行文件：
```bash
$ cd redis
$ sudo make
$ sudo make install
```

进入 `src/` 目录下可以查看 redis 的可执行文件：
```bash
$ cd src/
$ ll | grep redis-
-rwxr-xr-x. 1 root root 2077112 Dec 25 13:05 redis-benchmark
-rw-rw-r--. 1 root root   28351 Jan 25  2016 redis-benchmark.c
-rw-r--r--. 1 root root   97224 Dec 25 13:05 redis-benchmark.o
-rwxr-xr-x. 1 root root   24992 Dec 25 13:05 redis-check-aof
-rw-rw-r--. 1 root root    6328 Jan 25  2016 redis-check-aof.c
-rw-r--r--. 1 root root   33656 Dec 25 13:05 redis-check-aof.o
-rwxr-xr-x. 1 root root   55840 Dec 25 13:05 redis-check-dump
-rw-rw-r--. 1 root root   22274 Jan 25  2016 redis-check-dump.c
-rw-r--r--. 1 root root   70264 Dec 25 13:05 redis-check-dump.o
-rwxr-xr-x. 1 root root 2206728 Dec 25 13:05 redis-cli
-rw-rw-r--. 1 root root   75872 Jan 25  2016 redis-cli.c
-rw-r--r--. 1 root root  311552 Dec 25 13:05 redis-cli.o
-rwxr-xr-x. 1 root root 4359144 Dec 25 13:05 redis-sentinel
-rwxr-xr-x. 1 root root 4359144 Dec 25 13:05 redis-server
-rwxrwxr-x. 1 root root   60527 Jan 25  2016 redis-trib.rb
```

> redis-server: 可以启用一个redis服务器 <br>
> redis-cli: redis命令行客户端 <br>
> redis-benchmark: redis性能测试工具 <br>
> redis-check-aof: redis aof文件 修复工具，比如断电的时候文件可能存在损坏的情况 <br>
> redis-check-dump: redis rdb文件检查修复工具 <br>
> redis-sentinel: 启用redis-sentinel节点 <br>


## 启动 

### 最简启动

默认配置，非守护进程方式启动，日志是直接输出在终端：
```bash
$ pwd
/opt/soft/redis
$ redis-server
```

![](/img/redis/10.png)

第一句:
```bash
Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
```
这一句就告诉我们没有指定配置文件，使用默认配置。


第二句：
```bash
You requested maxclients of 10000 requiring at least 10032 max file descriptors.
```
这句让我们需要提高 `open files` 数量。


最后一句：
```bash
The server is now ready to accept connections on port 6379
```
这句话告诉我们启动的是 `6379` 端口。

当服务端启动之后，我们可以使用客户端进行连接。现在另开一个终端，使用Redis客户端进行连接，在任意目录下使用下命令即可对Redis进行连接，我这里使用的是虚拟机IP：
```bash
$ redis-cli -h 127.0.0.1 -p 6379
127.0.0.1:6379> ping
PONG
127.0.0.1:6379> set hello world
OK
127.0.0.1:6379> get hello
"world"
127.0.0.1:6379> exit
$
```

不指定 `-h` 默认的就是 `127.0.0.1` <br>
不指定 `-p` 默认的就是 `6379`

```bash
$ redis-cli
127.0.0.1:6379> del hello
(integer) 1
127.0.0.1:6379> get hello
(nil)
127.0.0.1:6379> exit
$
```

可以使用查看进程方式验证是否启动：
```shell
ps -ef | grep redis
netstat -antpl | grep redis
redis-cli -h ip -p port ping
```


### 动态参数启动

比如以 `6380` 端口启动 Redis:
```bash
$ redis-server --port 6380
```

![](/img/redis/20.png)

连接测试:
```bash
$ redis-cli -p 6380
127.0.0.1:6380> get hello
(nil)
127.0.0.1:6380> set hello world
OK
127.0.0.1:6380> get hello
"world"
127.0.0.1:6380> exit
$
```
 
查看Redis进程:
```bash
# ps -ef | grep redis-server | grep -v grep
root      1672  1600  2 10:57 pts/0    00:00:12 redis-server *:6379
root      1726  1701  2 11:03 pts/1    00:00:03 redis-server *:6380
$
```
可以看到有 6379 和 6380 在提供服务。

### 配置文件启动（推荐）

进入 `/opt/soft/redis` 目录，通常是在 redis 目录下新建一个 config 的配置目录，然后将 Redis 的默认配置拷贝到 config 目录:
```bash
# cd /opt/soft/redis
# sudo mkdir config
# sudo cp redis.conf config/
# cd config/
```

一台机器可能会部署多台Redis，一台机器上多个Redis就有可能涉及到很多的端口，配置文件也就会涉及到很多的端口，配置文件可以使用端口进行区分。

比如现在使用配置文件启动一个端口是 6381 的Redis服务：
```bash
# cd config/
# sudo mv redis.conf redis-6381.conf
#
```

查看下 redis-6381.conf 中默认配置，把所有的注释（#）和所有的空格（^$）去掉，就可以看到所有的配置了：
```bash
# cat redis-6381.conf | grep -v "#" | grep -v "^$"
daemonize no
pidfile /var/run/redis.pid
port 6379
tcp-backlog 511
timeout 0
tcp-keepalive 0
loglevel notice
logfile ""
databases 16
save 900 1
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename dump.rdb
dir ./
slave-serve-stale-data yes
slave-read-only yes
repl-diskless-sync no
repl-diskless-sync-delay 5
repl-disable-tcp-nodelay no
slave-priority 100
appendonly no
appendfilename "appendonly.aof"
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
lua-time-limit 5000
slowlog-log-slower-than 10000
slowlog-max-len 128
latency-monitor-threshold 0
notify-keyspace-events ""
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-entries 512
list-max-ziplist-value 64
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
hll-sparse-max-bytes 3000
activerehashing yes
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
hz 10
aof-rewrite-incremental-fsync yes
#
```

可以将上面的配置重定向到 `redis-6382.conf` 的文件中：
```bash
# sudo cat redis-6381.conf | grep -v "#" | grep -v "^$" > redis-6382.conf
```

接着将 6381.conf 文件删除:
```bash
# sudo rm -rf redis-6381.conf
```

daemonize no: 表示是否以守护进程方式启动，通常是以守护进程方式启动，所以设置为 yes <br>
pidfile /var/run/redis.pid: 表示进程号存储的位置，先不管，可以暂时删掉，这个其实也是可以使用端口号进行区别 <br>
port 6379: 端口号，这是需要设置为 6382 <br>

刚学，最开始先配置一下几个参数的配置，其他的参数可以暂时不用管，可以都删掉，整个文件只保留上面内容：
```bash
daemonize yes
port 6382
dir "/opt/soft/redis/data"
logfile "6382.log"
```

接着返回 redis 目录，并新建一个 data 目录：
```bash
# cd ..
# pwd
/opt/soft/redis
# sudo mkdir data
# sudo chmod 777 data/
# 
```

指定配置文件启动：
```bash
# redis-server config/redis-6382.conf
#
```

查看下进程是否存在:
```bash
# ps -ef | grep redis-server | grep 6382
root  21503     1  0 02:44 ?        00:00:00 redis-server *:6382
#
```

查看日志是存在:
```bash
# cd data/
# ll
total 4
-rw-rw-r--. 1 vagrant vagrant 2563 Dec 26 02:44 6382.log
# cat 6382.log
21503:M 26 Dec 02:44:07.327 # You requested maxclients of 10000 requiring at least 10032 max file descriptors.
21503:M 26 Dec 02:44:07.328 # Redis can't set maximum open files to 10032 because of OS error: Operation not permitted.
21503:M 26 Dec 02:44:07.328 # Current maximum open files is 4096. maxclients has been reduced to 4064 to compensate for low ulimit. If you need higher maxclients increase 'ulimit -n'.
                _._
           _.-``__ ''-._
      _.-``    `.  `_.  ''-._           Redis 3.0.7 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6382
 |    `-._   `._    /     _.-'    |     PID: 21503
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

21503:M 26 Dec 02:44:07.329 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
21503:M 26 Dec 02:44:07.329 # Server started, Redis version 3.0.7
21503:M 26 Dec 02:44:07.330 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
21503:M 26 Dec 02:44:07.330 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
21503:M 26 Dec 02:44:07.330 * The server is now ready to accept connections on port 6382
[vagrant@centos7-redis data]$
```

![](/img/redis/30.png)

### 建议

生产环境建议选择配置文件方式启动 ，因为生产环境通常在一个机器上部署多个redis, 而redis是单线程模型，现在服务器很多都是多核，通常为了资源的合理利用，通常一台机器部署很多redis，如果使用默认配置或者动态配置会很麻烦。所以推荐使用配置文件方式启动。
另外在使用配置文件方式启动时，单机多实例配置文件可以用端口区分开。 
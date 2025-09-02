+++
title = 'Redis Cluster搭建'
date = 2024-06-15T08:20:42+08:00
draft = false
categories = [ "Redis" ]
tags = [ "redis", "DBA" ]
+++

## 原生命令安装

### 步骤预览

**1、配置开启节点**

**2、meet**

**3、指派槽**

**4、主从关系分配**

### 安装操作

**节点配置**

进入 redis/config 目录，准备6个配置文件。

第一个配置文件redis-7000.conf: 
```bash
port 7000
daemonize yes
dir "/opt/redis/data"
logfile "7000.log"
dbfilename "dump-7000.rdb"
cluster-enabled yes
cluster-config-file nodes-7000.conf
cluster-require-full-coverage no
```

使用 `sed` 命令快速生成其他5个配置文件

```bash
# sed 's/7000/7001/g' redis-7000.conf > redis-7001.conf
# sed 's/7000/7002/g' redis-7000.conf > redis-7002.conf
# sed 's/7000/7003/g' redis-7000.conf > redis-7003.conf
# sed 's/7000/7004/g' redis-7000.conf > redis-7004.conf
# sed 's/7000/7005/g' redis-7000.conf > redis-7005.conf
```

启动：
```bash
# redis-server redis-7000.conf
# redis-server redis-7001.conf
# redis-server redis-7002.conf
# redis-server redis-7003.conf
# redis-server redis-7004.conf
# redis-server redis-7005.conf
# ps -ef | grep redis-
root        3233       1  0 17:47 ?        00:00:00 redis-server *:7000 [cluster]
root        3237       1  0 17:47 ?        00:00:00 redis-server *:7001 [cluster]
root        3241       1  0 17:47 ?        00:00:00 redis-server *:7002 [cluster]
root        3245       1  0 17:47 ?        00:00:00 redis-server *:7003 [cluster]
root        3249       1  0 17:47 ?        00:00:00 redis-server *:7004 [cluster]
root        3253       1  0 17:47 ?        00:00:00 redis-server *:7005 [cluster]
root        3259     301  0 17:48 pts/1    00:00:00 grep --color=auto redis-
```

连接任意一个节点，执行某个命令：
```bash
# redis-cli -p 7000
127.0.0.1:7000> set hello world
(error) CLUSTERDOWN The cluster is down
127.0.0.1:7000>
```

表示当前集群属于下线状态，不可用。集群模式下状态可用的标识是所有节点都分配了槽。

查看集群本地配置：
```bash
# cat /opt/redis/data/nodes-7000.conf
c3e020394968ea7a7ade871951d244d4db88b155 :0 myself,master - 0 0 0 connected
vars currentEpoch 0 lastVoteEpoch 0
```

也可以执行下面命令查看接节点信息，和上面结果差不多：
```bash
# redis-cli -p 7000 cluster nodes
c3e020394968ea7a7ade871951d244d4db88b155 :7000 myself,master - 0 0 0 connected
```

还可以查看集群信息：
```bash
# redis-cli -p 7000 cluster info
cluster_state:fail
cluster_slots_assigned:0
cluster_slots_ok:0
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:1
cluster_size:0
cluster_current_epoch:0
cluster_my_epoch:0
cluster_stats_messages_sent:0
cluster_stats_messages_received:0
```


**集群节点 meet 握手配置**

首先将 7000 节点和7001 节点进行 meet 操作，连接 7000 节点执行下面命令：
```bash
# redis-cli -p 7000 cluster meet 127.0.0.1 7001
OK
```

此时在 7000 或者 7001 节点查看nodes信息，都已经达成了握手，其他节点并没有完成握手：
```bash
# redis-cli -p 7000 cluster nodes
c3e020394968ea7a7ade871951d244d4db88b155 127.0.0.1:7000 myself,master - 0 0 0 connected
820b55158df9acd4c3e41c758bdcabc6b0e4509b 127.0.0.1:7001 master - 0 1732010290107 1 connected
# redis-cli -p 7001 cluster nodes
c3e020394968ea7a7ade871951d244d4db88b155 127.0.0.1:7000 master - 0 1732010321733 0 connected
820b55158df9acd4c3e41c758bdcabc6b0e4509b 127.0.0.1:7001 myself,master - 0 0 1 connected
# redis-cli -p 7002 cluster nodes
c074b7dee3455cf9357c99fb3aae6be1992edfdf :7002 myself,master - 0 0 0 connected
```

然后再继续执行其他节点的 meet 操作：
```bash
# redis-cli -p 7000 cluster meet 127.0.0.1 7002
OK
# redis-cli -p 7000 cluster meet 127.0.0.1 7003
OK
# redis-cli -p 7000 cluster meet 127.0.0.1 7004
OK
# redis-cli -p 7000 cluster meet 127.0.0.1 7005
OK
```

在7000上查看nodes和cluster信息：
```bash
# redis-cli -p 7000 cluster nodes
c3e020394968ea7a7ade871951d244d4db88b155 127.0.0.1:7000 myself,master - 0 0 3 connected
820b55158df9acd4c3e41c758bdcabc6b0e4509b 127.0.0.1:7001 master - 0 1732010453751 1 connected
e3eec80472f946286c0fbf7e7270ecce6549adf8 127.0.0.1:7004 master - 0 1732010455776 0 connected
b44eba6f9e3db1d78220e565e67bfdd42d61505d 127.0.0.1:7005 master - 0 1732010458816 5 connected
c074b7dee3455cf9357c99fb3aae6be1992edfdf 127.0.0.1:7002 master - 0 1732010457805 2 connected
218afab543610e66b74922975ea38710649f2f74 127.0.0.1:7003 master - 0 1732010459850 4 connected
# redis-cli -p 7000 cluster info
cluster_state:fail
cluster_slots_assigned:0
cluster_slots_ok:0
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:0
cluster_current_epoch:5
cluster_my_epoch:3
cluster_stats_messages_sent:556
cluster_stats_messages_received:556
```

此时到这里集群节点已经达到了互通的效果。但此时集群的状态仍然是不可用，所以接下来需要为集群分配槽。

**分配槽**

编写脚本 addslot.sh：
```bash
# 起始slot
start=$1
# 终止slot
end=$2

# 端口
port=$3

for slot in `seq ${start} ${end}`
do
    echo "slot: ${slot}"
    redis-cli -p ${port} cluster addslots ${slot}
done
```

执行脚本第一个节点分派槽：
```bash
sh addslots.sh 0 5461 7000
```

查看第一个7000节点的集群和节点信息，会看到分配5462个槽：
```bash
# redis-cli -p 7000 cluster info
cluster_state:ok
cluster_slots_assigned:5462
cluster_slots_ok:5462
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:1
cluster_current_epoch:5
cluster_my_epoch:3
cluster_stats_messages_sent:9448
cluster_stats_messages_received:9448
# redis-cli -p 7000 cluster nodes
c3e020394968ea7a7ade871951d244d4db88b155 127.0.0.1:7000 myself,master - 0 0 3 connected 0-5461
820b55158df9acd4c3e41c758bdcabc6b0e4509b 127.0.0.1:7001 master - 0 1732026371394 1 connected
e3eec80472f946286c0fbf7e7270ecce6549adf8 127.0.0.1:7004 master - 0 1732026369367 0 connected
b44eba6f9e3db1d78220e565e67bfdd42d61505d 127.0.0.1:7005 master - 0 1732026372407 5 connected
c074b7dee3455cf9357c99fb3aae6be1992edfdf 127.0.0.1:7002 master - 0 1732026370380 2 connected
218afab543610e66b74922975ea38710649f2f74 127.0.0.1:7003 master - 0 1732026368356 4 connected
```

接着给其余的节点分配槽：
```bash
sh addslots.sh 5462 10922 7001
sh addslots.sh 10923 16383 7002
```

此时再看下集群的状态ok，也已经分配了16384个槽：
```bash
redis-cli -p 7000 cluster info
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:5
cluster_my_epoch:3
cluster_stats_messages_sent:11727
cluster_stats_messages_received:11727
# redis-cli -p 7000 cluster nodes
c3e020394968ea7a7ade871951d244d4db88b155 127.0.0.1:7000 myself,master - 0 0 3 connected 0-5461
820b55158df9acd4c3e41c758bdcabc6b0e4509b 127.0.0.1:7001 master - 0 1732028930955 1 connected 5462-10922
e3eec80472f946286c0fbf7e7270ecce6549adf8 127.0.0.1:7004 master - 0 1732028928928 0 connected
b44eba6f9e3db1d78220e565e67bfdd42d61505d 127.0.0.1:7005 master - 0 1732028931968 5 connected
c074b7dee3455cf9357c99fb3aae6be1992edfdf 127.0.0.1:7002 master - 0 1732028932375 2 connected 10923-16383
218afab543610e66b74922975ea38710649f2f74 127.0.0.1:7003 master - 0 1732028930448 4 connected
````

最后一步分配主从关系，7003节点为7000的从节点，7004节点为7001的从节点，7005为7002的从节点。

比如让7003作为从，就连接到7003节点执行下面命令：
```bash
# redis-cli -p 7003 cluster replicate c3e020394968ea7a7ade871951d244d4db88b155
OK
```

然后查看下节点信息：
```bash
# redis-cli -p 7000 cluster nodes
c3e020394968ea7a7ade871951d244d4db88b155 127.0.0.1:7000 myself,master - 0 0 3 connected 0-5461
820b55158df9acd4c3e41c758bdcabc6b0e4509b 127.0.0.1:7001 master - 0 1732030290643 1 connected 5462-10922
e3eec80472f946286c0fbf7e7270ecce6549adf8 127.0.0.1:7004 master - 0 1732030287606 0 connected
b44eba6f9e3db1d78220e565e67bfdd42d61505d 127.0.0.1:7005 master - 0 1732030291656 5 connected
c074b7dee3455cf9357c99fb3aae6be1992edfdf 127.0.0.1:7002 master - 0 1732030289630 2 connected 10923-16383
218afab543610e66b74922975ea38710649f2f74 127.0.0.1:7003 slave c3e020394968ea7a7ade871951d244d4db88b155 0 1732030292668 4 connected
```

同理设置其他两个节点的主从关系：
```bash
# redis-cli -p 7004 cluster replicate 820b55158df9acd4c3e41c758bdcabc6b0e4509b
OK
# redis-cli -p 7005 cluster replicate c074b7dee3455cf9357c99fb3aae6be1992edfdf
OK
```

再次查看节点信息，发现主从已全部分配：
```bash
# redis-cli -p 7000 cluster nodes
c3e020394968ea7a7ade871951d244d4db88b155 127.0.0.1:7000 myself,master - 0 0 3 connected 0-5461
820b55158df9acd4c3e41c758bdcabc6b0e4509b 127.0.0.1:7001 master - 0 1732030438345 1 connected 5462-10922
e3eec80472f946286c0fbf7e7270ecce6549adf8 127.0.0.1:7004 slave 820b55158df9acd4c3e41c758bdcabc6b0e4509b 0 1732030438852 1 connected
b44eba6f9e3db1d78220e565e67bfdd42d61505d 127.0.0.1:7005 slave c074b7dee3455cf9357c99fb3aae6be1992edfdf 0 1732030440884 5 connected
c074b7dee3455cf9357c99fb3aae6be1992edfdf 127.0.0.1:7002 master - 0 1732030439866 2 connected 10923-16383
218afab543610e66b74922975ea38710649f2f74 127.0.0.1:7003 slave c3e020394968ea7a7ade871951d244d4db88b155 0 1732030440377 4 connected
```

另外还使用下面查看槽分配信息：
```bash
# redis-cli -p 7000 cluster slots
1) 1) (integer) 0
   2) (integer) 5461
   3) 1) "127.0.0.1"
      2) (integer) 7000
   4) 1) "127.0.0.1"
      2) (integer) 7003
2) 1) (integer) 5462
   2) (integer) 10922
   3) 1) "127.0.0.1"
      2) (integer) 7001
   4) 1) "127.0.0.1"
      2) (integer) 7004
3) 1) (integer) 10923
   2) (integer) 16383
   3) 1) "127.0.0.1"
      2) (integer) 7002
   4) 1) "127.0.0.1"
      2) (integer) 7005
```

最后还要做一个关于数据的操作：
```bash
# redis-cli -c -p 7000
127.0.0.1:7000> set hello world
OK
```

注意：生产环境不会采用这种方式进行安装，但这有利于理解 redis cluster 集群。

## 官方工具安装

- 下载、编译、安装ruby
- 安装rubygem redis
- 安装redis-trib.rb


1、下载安装
```bash
wget https://cache.ruby-lang.org/pub/ruby/2.3/ruby-2.3.1.tar.gz
tar -zxvf ruby-2.3.1.tar.gz
cd ruby-2.3.1
./configure --prefix=/usr/local/ruby
make && make install
cp /usr/local/ruby/bin/ruby /usr/local/bin
cp /usr/local/ruby/bin/gem /usr/local/bin
```

2、安装rubygem redis
```bash
wget https://rubygems.org/downloads/redis-3.3.0.gem
gem install -l redis-3.3.0.gem
gem list -- check redis gem
```

3、安装redis-trib.tb
```bash
cp ${REDIS_HOME}/src/redis-trib.rb /usr/local/bin
```

进入 redis/src/
```bash
# ./redis-trib.rb
Usage: redis-trib <command> <options> <arguments ...>

  create          host1:port1 ... hostN:portN
                  --replicas <arg>
  check           host:port
  info            host:port
  fix             host:port
                  --timeout <arg>
  reshard         host:port
                  --from <arg>
                  --to <arg>
                  --slots <arg>
                  --yes
                  --timeout <arg>
                  --pipeline <arg>
  rebalance       host:port
                  --weight <arg>
                  --auto-weights
                  --use-empty-masters
                  --timeout <arg>
                  --simulate
                  --pipeline <arg>
                  --threshold <arg>
  add-node        new_host:new_port existing_host:existing_port
                  --slave
                  --master-id <arg>
  del-node        host:port node_id
  set-timeout     host:port milliseconds
  call            host:port command arg arg .. arg
  import          host:port
                  --from <arg>
                  --copy
                  --replace
  help            (show this help)

For check, fix, reshard, del-node, set-timeout you can specify the host and port of any working node in the cluster.
```

接着使用 redis-trib 做集群的搭建。

批量杀死进程：
```bash
ps -ef | grep redis-server | grep 700 | awk '{print $2}' | xargs kill
```

然后将之前 data 目录清理。
```bash
rm -rf /opt/redis/data/*
```

节点配置和以前一样，保持不变。比如redis-7000.conf配置如下，其他节点除了端口其他一致：
```bash
# cat redis-7000.conf
port 7000
daemonize yes
dir "/opt/redis/data"
logfile "7000.log"
dbfilename "dump-7000.rdb"
cluster-enabled yes
cluster-config-file nodes-7000.conf
cluster-require-full-coverage no
```

启动节点：
```bash
redis-server redis-7000.conf
redis-server redis-7001.conf
redis-server redis-7002.conf
redis-server redis-7003.conf
redis-server redis-7004.conf
redis-server redis-7005.conf
```

redis-trib 安装集群：
```bash
cd ../src
./redis-trib.rb create --replicas 1 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005
```

完整过程如下：
```bash
# ./redis-trib.rb create --replicas 1 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005
>>> Creating cluster
>>> Performing hash slots allocation on 6 nodes...
Using 3 masters:
127.0.0.1:7000
127.0.0.1:7001
127.0.0.1:7002
Adding replica 127.0.0.1:7003 to 127.0.0.1:7000
Adding replica 127.0.0.1:7004 to 127.0.0.1:7001
Adding replica 127.0.0.1:7005 to 127.0.0.1:7002
M: f2c72a49dc3f0640154b8018f6ee181f74bd63b6 127.0.0.1:7000
   slots:0-5460 (5461 slots) master
M: 09a9e52f1def7d9c56bb824b3e06dd3b6be712e8 127.0.0.1:7001
   slots:5461-10922 (5462 slots) master
M: a97e3ac902b2ee221feb296e6bfe38fc1f012bf5 127.0.0.1:7002
   slots:10923-16383 (5461 slots) master
S: 203727f14f10c21943ef8262d77455b6925b33b6 127.0.0.1:7003
   replicates f2c72a49dc3f0640154b8018f6ee181f74bd63b6
S: 5cffa2e7fb048b2a6ba3d3e568a9d2b49c78e46d 127.0.0.1:7004
   replicates 09a9e52f1def7d9c56bb824b3e06dd3b6be712e8
S: 95da5dc272e79f4756ba258308dc8fb73aafaad0 127.0.0.1:7005
   replicates a97e3ac902b2ee221feb296e6bfe38fc1f012bf5
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join.....
>>> Performing Cluster Check (using node 127.0.0.1:7000)
M: f2c72a49dc3f0640154b8018f6ee181f74bd63b6 127.0.0.1:7000
   slots:0-5460 (5461 slots) master
M: 09a9e52f1def7d9c56bb824b3e06dd3b6be712e8 127.0.0.1:7001
   slots:5461-10922 (5462 slots) master
M: a97e3ac902b2ee221feb296e6bfe38fc1f012bf5 127.0.0.1:7002
   slots:10923-16383 (5461 slots) master
M: 203727f14f10c21943ef8262d77455b6925b33b6 127.0.0.1:7003
   slots: (0 slots) master
   replicates f2c72a49dc3f0640154b8018f6ee181f74bd63b6
M: 5cffa2e7fb048b2a6ba3d3e568a9d2b49c78e46d 127.0.0.1:7004
   slots: (0 slots) master
   replicates 09a9e52f1def7d9c56bb824b3e06dd3b6be712e8
M: 95da5dc272e79f4756ba258308dc8fb73aafaad0 127.0.0.1:7005
   slots: (0 slots) master
   replicates a97e3ac902b2ee221feb296e6bfe38fc1f012bf5
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

- `1` 表示为每个主节点设置一个从节点

查看集群信息：
```bash
# redis-cli -p 7000 cluster info
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:6
cluster_my_epoch:1
cluster_stats_messages_sent:430
cluster_stats_messages_received:430

# redis-cli -p 7000 cluster nodes
09a9e52f1def7d9c56bb824b3e06dd3b6be712e8 127.0.0.1:7001 master - 0 1732064983772 2 connected 5461-10922
5cffa2e7fb048b2a6ba3d3e568a9d2b49c78e46d 127.0.0.1:7004 slave 09a9e52f1def7d9c56bb824b3e06dd3b6be712e8 0 1732064982760 5 connected
95da5dc272e79f4756ba258308dc8fb73aafaad0 127.0.0.1:7005 slave a97e3ac902b2ee221feb296e6bfe38fc1f012bf5 0 1732064985792 6 connected
a97e3ac902b2ee221feb296e6bfe38fc1f012bf5 127.0.0.1:7002 master - 0 1732064984783 3 connected 10923-16383
203727f14f10c21943ef8262d77455b6925b33b6 127.0.0.1:7003 slave f2c72a49dc3f0640154b8018f6ee181f74bd63b6 0 1732064983265 4 connected
f2c72a49dc3f0640154b8018f6ee181f74bd63b6 127.0.0.1:7000 myself,master - 0 0 1 connected 0-5460
```

**注意**

安装ruby依赖zlib, openssl 等依赖。

## 总结

**原生命令安装**

- 可以帮助我们理解 Redis Cluster 架构
- 生产环境一般不使用

**ruby-trib官方工具安装**

- 高效、准确、易于使用
- 生产环境可以使用

**其他 **
可视化部署

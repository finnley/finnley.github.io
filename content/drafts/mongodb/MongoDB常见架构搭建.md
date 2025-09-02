+++
title = 'MongoDB常见架构搭建'
date = 2024-05-12T07:23:05+08:00
draft = true
categories = [ "MongoDB" ]
+++

# 架构

* 单实例架构搭建
* 副本集架构搭建
* 分片架构搭建

# 准备

1. [下载地址](https://www.mongodb.com/try/download/community)
2. 选择操作系统和版本
3. 获取 [下载链接](https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel70-5.0.28.tgz)

快捷下载：
```bash
wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel70-5.0.28.tgz
```

# 单实例架构搭建

1、解压
```bash
tar zxvf mongodb-linux-x86_64-rhel70-5.0.28.tgz -C /opt
```

2、创建数据目录和日志目录
```bash
mkdir -p /data/mongodb/{db,log}
```

3、添加环境变量
```bash
export PATH=$PATH:/opt/mongodb-linux-x86_64-rhel70-5.0.28/bin/
```

4、启动
```bash
mongod --dbpath /data/mongodb/db --logpath /data/mongodb/log/mongod.log --fork --bind_ip 0.0.0.0
```

- **mongod**：启动服务命令。
- **--dbpath**：后接数据目录。
- **--logpath**：后接日志文件。
- **--fork**：表示以后台 `daemon` 方式运行。
- **--bind_ip**：表示对外服务 IP，`0.0.0.0` 表示任何IP都可以连接 MongoDB 服务。

输出内容如下：
```bash
about to fork child process, waiting until server is ready for connections.
forked process: 322
child process started successfully, parent exiting
```

5、验证
```bash
ps -ef | grep mongo
```

输出内容如下：
```bash
root       322     0  0 23:58 ?        00:00:00 mongod --dbpath /data/mongodb/db --logpath /data/mongodb/log/mongod.log --fork --bind_ip 0.0.0.0
root       396   282  0 23:59 pts/1    00:00:00 grep --color=auto mongo
```

6、登录
```bash
mongo
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
```

# 副本集架构搭建

### 架构

![](/img/mongodb/replica/190.png)

常见的架构如上图，`Secondary` 节点持续复制 `Primary` 节点的数据，`Arbiter` 节点负责心跳检测和选举，该节点不接收数据。

### 副本集复制原理

当 Primary 节点有修改操作时，会将变更记录在 `oplog` 中，有一个线程会持续监听 `oplog` 的变化，当有变化时会将变化传到 `Secondary` 从节点，然后在从节点回放。

### 故障恢复原理

副本集中节点两两互相发送心跳，当超过5次未收到对方心跳时，就会认为对方失联，如果失联的是主节点，那么从节点会发起选举，选举出新的主节点，如果失联的是从节点，则不会发起新的选举，选举时基于raft一致性算法。

### 副本集部署

1、准备3个节点，分别是 Primary、Secondary、Arbiter 节点。

2、下载、解压

3、三台机器节点分别创建数据目录。
```bash
mkdir -p /data/{mongodb27001/data/configdb,mongodb27001/data/sharddb,mongodb27001/conf,mongodb27001/run,mongodb27001/logs}
```

4、三台机器分别设置环境变量
```bash
export PATH=$PATH:/opt/mongodb-linux-x86_64-rhel70-5.0.27/bin/
```

5、三台机器分别修改安装配置文件
```bash
vim /data/mongodb27001/conf/mongod.conf
```

完整配置如下：
```bash
systemLog:
  destination: file
  logAppend: true
  path: /data/mongodb27001/logs/mongod.log
storage:
  dbPath: /data/mongodb27001/data
processManagement:
  fork: true
  pidFilePath: /data/mongodb27001/run/mongod.pid
net:
  port: 27001
  bindIp: 0.0.0.0
replication:
  replSetName: testdb_repl
```

```bash
systemLog:
  destination: file # 日志形式
  logAppend: true # 日志追加方式
  path: /data/mongodb27001/logs/mongod.log # 日志目录
storage:
  dbPath: /data/mongodb27001/data
processManagement:
  fork: true # true表示以后台方式运行
  pidFilePath: /data/mongodb27001/run/mongod.pid
net:
  port: 27001
  bindIp: 0.0.0.0 # 所有机器都可以访问MongoDB
replication:
  replSetName: testdb_repl # 副本集名称
```

5、三台机器分别启动mongodb
```bash
mongod -f /data/mongodb27001/conf/mongod.conf
```

6、创建集群
第一台机器：
```bash
mongo --port 27001
```

添加集群配置：
```mongo
config={
    _id: 'testdb_repl', 
    members: [
        {_id: 0, host: '172.10.20.1:27001', priority: 1}, 
        {_id: 1, host: '172.10.20.2:27001', priority: 1}, 
        {_id: 2, host: '172.10.20.3:27001', arbiterOnly: true}, 
    ]
}
```

7、初始化配置：
```mongo
rs.initiate(config)
```

不断回车后看到第一个节点就是Primary 节点。
```bash
> rs.initiate(config)
{ "ok" : 1 }
testdb_repl:SECONDARY> 
testdb_repl:SECONDARY> 
testdb_repl:PRIMARY> 
testdb_repl:PRIMARY> 
```


8、查看集群状态：
```moingo
rs.status()
```

9、数据同步验证，进入 Secondary 节点：
```mongo
testdb_repl:SECONDARY> use testdb
switched to db testdb
testdb_repl:SECONDARY> db.goods.find()
Error: error: {
        "topologyVersion" : {
                "processId" : ObjectId("6680fbab240af9d874708ab8"),
                "counter" : NumberLong(4)
        },
        "ok" : 0,
        "errmsg" : "not master and slaveOk=false",
        "code" : 13435,
        "codeName" : "NotPrimaryNoSecondaryOk",
        "$clusterTime" : {
                "clusterTime" : Timestamp(1719729498, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        },
        "operationTime" : Timestamp(1719729498, 1)
}
testdb_repl:SECONDARY> 
```

这里“not master and slaveOk=false” 提示该节点不是主节点。通常 Secondary节点需要设置才能提供查询：
```bash
testdb_repl:SECONDARY> rs.secondaryOk()
testdb_repl:SECONDARY> db.goods.find()
testdb_repl:SECONDARY>
```

说明在 testdb_repl 这个database 里面的集合 goods 中没有数据


接着进入 primary 写入数据
```mongo
testdb_repl:PRIMARY> use testdb
switched to db testdb
testdb_repl:PRIMARY> db.goods.save({name:"zhangsan"})
WriteResult({ "nInserted" : 1 })
testdb_repl:PRIMARY> db.goods.find()
{ "_id" : ObjectId("6680fe78c3e5906aa8903e5a"), "name" : "zhangsan" }
testdb_repl:PRIMARY> 
```

再到secondary节点查询：
```mongo
testdb_repl:SECONDARY> db.goods.find()
{ "_id" : ObjectId("6680fe78c3e5906aa8903e5a"), "name" : "zhangsan" }
testdb_repl:SECONDARY> 
```

发现数据已经从 primary 节点同步到 secondary 节点了。

# 副本集架构搭建

### 使用场景

* 数据量过大，单机备份和恢复耗时长
* 副本集已无法满足高并发写入的性能要求

### 特点

* 对应用透明，应用通过分片方式连接到分片集群，与普通单节点连接一样
* 数据自动平衡，不会出现时间长数据在各分片不均匀情况
* 支持动态扩容，扩容期间MongoDB可继续提供服务，不必停机

### 架构

![](/img/mongodb/sharding/30.png)

Shard 用来存储数据，为了提高高可用和数据一致性，一个分片就是一个副本集，比如图中的 shard1 和shard2，分片之间的数据是不重复的。

Router（mongos）节点与客户端连接，并将操作定向到一个或多个分片中，生产环境可以配置多个 mongos 来实现高可用和负载均衡

Config Server 节点用来存储集群的元数据，包括集群数据集到分片的映射。Router 根据元数据将操作定向到对应的分片中。


### 部署

1、在三台机器上各启动一个 config server，它们的端口都是27020，这些 config server 会配置为一个副本集。

2、分别在3台机器上启动 27001 和 27002 端口，三台机器上的 27001 端口对应的第一个副本集，作为第一个分片，三台机器上的 27002 端口对应的第二个副本集，作为第二个分片。

#### config server

三台机器各自执行下面命令，不同的是 bing_ip 需要修改为各自机器的IP：

第一台：
```bash
mkdir -p /data/mongodb/config
mongod --configsvr --replSet config --dbpath /data/mongodb/config --bind_ip localhost,172.10.20.1 --logpath /data/mongodb/config/mongod.log --port 27020 --fork
```

configsvr: 表示这个节点是配置服务器
replSet: 配置的集合名称
dbpath: 表示配置的是数据目录
bind_ip: 机器IP
logpath: 日志文件路径
fork: 以后台进程方式启动

第二台：
```bash
mkdir -p /data/mongodb/config
mongod --configsvr --replSet config --dbpath /data/mongodb/config --bind_ip localhost,172.10.20.2 --logpath /data/mongodb/config/mongod.log --port 27020 --fork
```

第三台：
```bash
mkdir -p /data/mongodb/config
mongod --configsvr --replSet config --dbpath /data/mongodb/config --bind_ip localhost,172.10.20.3 --logpath /data/mongodb/config/mongod.log --port 27020 --fork
```

连接其中一台机器，我连接的第一台机器，创建 config server 的副本集。
```bash
mongo --port 27020
> rs.initiate(
    {
        _id: "config",
        configsvr: true,
        members: [
            {_id: 0, host: "172.10.20.1:27020"},
            {_id: 1, host: "172.10.20.2:27020"},
            {_id: 2, host: "172.10.20.3:27020"}
        ]
    }
)
回车
回车
config:SECONDARY> 
config:SECONDARY> 
config:SECONDARY> 
config:PRIMARY> 
config:PRIMARY>
```

查看副本集状态：
```mongo
> rs.status()
```

关注 members，会有一个 Primary 和两个 Secondary，health 都是1表示集群创建爱你成功。


#### Sharding

启动mongodb实例，这些用来存储数据的分片mongodb，在一台机器上部署两个实例，我在第一台机器上执行：
```bash
mkdir -p /data/mongodb/shardtest01
mkdir -p /data/mongodb/shardtest02

mongod --shardsvr --replSet shardtest01 --dbpath /data/mongodb/shardtest01 --bind_ip localhost,172.10.20.1 --logpath /data/mongodb/shardtest01/mongod.log --port 27001 --fork

mongod --shardsvr --replSet shardtest02 --dbpath /data/mongodb/shardtest02 --bind_ip localhost,172.10.20.1 --logpath /data/mongodb/shardtest02/mongod.log --port 27002 --fork
```

第二机器创建也部署两个实例：
```bash
mkdir -p /data/mongodb/shardtest01
mkdir -p /data/mongodb/shardtest02

mongod --shardsvr --replSet shardtest01 --dbpath /data/mongodb/shardtest01 --bind_ip localhost,172.10.20.2 --logpath /data/mongodb/shardtest01/mongod.log --port 27001 --fork

mongod --shardsvr --replSet shardtest02 --dbpath /data/mongodb/shardtest02 --bind_ip localhost,172.10.20.2 --logpath /data/mongodb/shardtest02/mongod.log --port 27002 --fork
```

第三机器创建也部署两个实例：
```bash
mkdir -p /data/mongodb/shardtest01
mkdir -p /data/mongodb/shardtest02

mongod --shardsvr --replSet shardtest01 --dbpath /data/mongodb/shardtest01 --bind_ip localhost,172.10.20.3 --logpath /data/mongodb/shardtest01/mongod.log --port 27001 --fork

mongod --shardsvr --replSet shardtest02 --dbpath /data/mongodb/shardtest02 --bind_ip localhost,172.10.20.3 --logpath /data/mongodb/shardtest02/mongod.log --port 27002 --fork
```

shardsvr: 表示该节点为分片节点，分片副本集
replSet: 副本集名称

查看每台机器mongodb启动情况：
每台机器
```bash
ps -ef | grep mongo
```

#### 创建分片副本集

第一个副本集机器
```bash
mongo --port 27001
> rs.initiate(
    {
        _id: "shardtest01",
        members: [
            {_id: 0, host: "172.10.20.1:27001"},
            {_id: 1, host: "172.10.20.2:27001"},
            {_id: 2, host: "172.10.20.3:27001"}
        ]
    }
)
> rs.status()
```

第二个副本集机器
```bash
mongo --port 27002
> rs.initiate(
    {
        _id: "shardtest02",
        members: [
            {_id: 0, host: "172.10.20.1:27002"},
            {_id: 1, host: "172.10.20.2:27002"},
            {_id: 2, host: "172.10.20.3:27002"}
        ]
    }
)
> rs.status()
```

#### mongos

启动mongos，随便选择一台机器启动mongos，生产环境一般建议在业务机器启动机客户端启动：
```bash
mkdir -p /data/mongodb/mongos
# 配置 config server 集群
mongos --configdb config/172.10.20.1:27020,172.10.20.2:27020,172.10.20.3:27020 --bind_ip localhost,172.10.20.1 --logpath /data/mongodb/mongos/mongos.log --port 27017 --fork
```

bind_ip 为对客户端开放的IP

#### 将分片副本集添加到集群

连接mongos：
```mongo
mongo --port 27017
> sh.addShard("shardtest01/172.10.20.1:27001,172.10.20.2:27001,172.10.20.3:27001")
> sh.addShard("shardtest02/172.10.20.1:27002,172.10.20.2:27002,172.10.20.3:27002")
```

创建分片表：
```mongo
> mongo --port 27017
sh.enableSharding("testdb") # 为数据库 testdb 启动分片
```

对集合创建分片规则：
```mongo
mongo --port 27017
# 分片字段为_id， 分片方法为哈希
sh.shardCollection("testdb.userinfo", {_id: "hashed"})
> sh.status()
```

```mongos
mongos> sh.status()
--- Sharding Status --- 
  sharding version: {
        "_id" : 1,
        "minCompatibleVersion" : 5,
        "currentVersion" : 6,
        "clusterId" : ObjectId("6681032ef5404a8b857984ff")
  }
  shards:
        {  "_id" : "shardtest01",  "host" : "shardtest01/172.10.20.1:27001,172.10.20.2:27001,172.10.20.3:27001",  "state" : 1,  "topologyTime" : Timestamp(1719731862, 2) }
        {  "_id" : "shardtest02",  "host" : "shardtest02/172.10.20.1:27002,172.10.20.2:27002,172.10.20.3:27002",  "state" : 1,  "topologyTime" : Timestamp(1719731873, 1) }
  active mongoses:
        "5.0.27" : 1
  autosplit:
        Currently enabled: yes
  balancer:
        Currently enabled: yes
        Currently running: no
        Failed balancer rounds in last 5 attempts: 0
        Migration results for the last 24 hours: 
                No recent migrations
  databases:
        {  "_id" : "config",  "primary" : "config",  "partitioned" : true }
        {  "_id" : "testdb",  "primary" : "shardtest02",  "partitioned" : true,  "version" : {  "uuid" : UUID("ed69bcb9-8210-4216-84c0-bc6148449ef8"),  "timestamp" : Timestamp(1719731906, 1),  "lastMod" : 1 } }
                testdb.userinfo
                        shard key: { "_id" : "hashed" }
                        unique: false
                        balancing: true
                        chunks:
                                shardtest01     2
                                shardtest02     2
                        { "_id" : { "$minKey" : 1 } } -->> { "_id" : NumberLong("-4611686018427387902") } on : shardtest01 Timestamp(1, 0) 
                        { "_id" : NumberLong("-4611686018427387902") } -->> { "_id" : NumberLong(0) } on : shardtest01 Timestamp(1, 1) 
                        { "_id" : NumberLong(0) } -->> { "_id" : NumberLong("4611686018427387902") } on : shardtest02 Timestamp(1, 2) 
                        { "_id" : NumberLong("4611686018427387902") } -->> { "_id" : { "$maxKey" : 1 } } on : shardtest02 Timestamp(1, 3) 
mongos> 
```

注意如果userinfo drop重建就需要重建collection分片规则。

#### 分片集群数据分布测试

```mongo
mongo --port 27017 testdb

> for (var i = 1; i <=10; i++) db.userinfo.save({userid: i, username: "aa"})

> db.userinfo.find()
```

首先通过mongos查看两个分片的分片情况：
先登录第一个shard：
```mongo
mongo --port 27001 testdb
> db.userinfo.find()
shardtest01:PRIMARY> db.userinfo.find()
{ "_id" : ObjectId("6681073bf566863202d3a6cb"), "userid" : 2, "username" : "aa" }
{ "_id" : ObjectId("6681073bf566863202d3a6cc"), "userid" : 3, "username" : "aa" }
{ "_id" : ObjectId("6681073bf566863202d3a6ce"), "userid" : 5, "username" : "aa" }
{ "_id" : ObjectId("6681073bf566863202d3a6d0"), "userid" : 7, "username" : "aa" }
{ "_id" : ObjectId("6681073bf566863202d3a6d1"), "userid" : 8, "username" : "aa" }
{ "_id" : ObjectId("6681073bf566863202d3a6d2"), "userid" : 9, "username" : "aa" }
shardtest01:PRIMARY>
```

先登录第二个shard：
```mongo
mongo --port 27002 testdb
> db.userinfo.find()
shardtest02:PRIMARY> db.userinfo.find()
{ "_id" : ObjectId("6681073bf566863202d3a6ca"), "userid" : 1, "username" : "aa" }
{ "_id" : ObjectId("6681073bf566863202d3a6cd"), "userid" : 4, "username" : "aa" }
{ "_id" : ObjectId("6681073bf566863202d3a6cf"), "userid" : 6, "username" : "aa" }
{ "_id" : ObjectId("6681073bf566863202d3a6d3"), "userid" : 10, "username" : "aa" }
shardtest02:PRIMARY>
```



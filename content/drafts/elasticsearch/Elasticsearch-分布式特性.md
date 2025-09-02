+++
title = 'Elasticsearch分布式特性'
date = 2020-05-31T16:48:55+08:00
draft = true
categories = [ "ElasticSearch" ]
tags = [ "es" ]
+++

## 分布式特性

es 支持集群模式，是一个分布式系统

* 优点

1. 增大系统容量，如内存，磁盘，使得es集群可以支持PB级的数据

2. 提高系统可用性，即使部分节点停止服务，整个集群依然可以正常服务

es 集群由多个es实例组成

1. 不同集群通过集群名字来区分，可以通过 cluster.name 进行修改，默认为 elasticsearch

2. 每个 es 实例本质上是一个 JVM  进程，且有自己的名字，通过 Node.name 进行修改

#### cerebro

github 地址

[cerebro](https://github.com/lmenezes/cerebro "cerebro")

![](https://images.notes.xuepincat.com/elastic/elasticsearch/55.png)


点击 `releases`

![](https://images.notes.xuepincat.com/elastic/elasticsearch/56.png)

下载

![](https://images.notes.xuepincat.com/elastic/elasticsearch/57.png)

我下载的 `tgz` 格式的，解压

```
tar zxvf cerebro-0.9.1.tgz
cd cerebro-0.9.1
ll
```

![](https://images.notes.xuepincat.com/elastic/elasticsearch/58.png)

运行

```
bin/cerebro
```

![](https://images.notes.xuepincat.com/elastic/elasticsearch/59.png)

默认端口 `9000`,我访问 `127.0.0.1:9000` 尽然访问不了，需要访问 `localhost:9000`

![](https://images.notes.xuepincat.com/elastic/elasticsearch/60.png)

在文本框中输入要访问的地址，这里我输入的本地的9200端口，然后点击 `Connect`

![](https://images.notes.xuepincat.com/elastic/elasticsearch/61.png)

看到下面页面就表示连接成功

![](https://images.notes.xuepincat.com/elastic/elasticsearch/62.png)

## 构建集群

#### 启动一个节点

运行下面命令可以启动一个es节点实例

```
bin/elasticsearch -Ecluster.name=my_cluster -Epath.data=my_cluster_node1 -E node.name=node1 -Ehttp.port=5200 -d
```
`-Epath.data` 不同的node使用不同的 path.data
`-d` 表示在后台以进程方式运行

![](https://images.notes.xuepincat.com/elastic/elasticsearch/63.png)

打开cerebro，点击有右上角

![](https://images.notes.xuepincat.com/elastic/elasticsearch/64.png)

输入访问的地址

![](https://images.notes.xuepincat.com/elastic/elasticsearch/65.png)

出现了下面页面表示连接成功

![](https://images.notes.xuepincat.com/elastic/elasticsearch/66.png)

现在里面是一个所有都没有的

#### Cluster State

es 集群相关的数据称为 `cluster state`,主要记录如下信息：

1. 节点信息，比如节点名称，连接地址等

2. 索引信息，比如索引名称，配置等

#### Master Node

* 可以修改 cluster state 的节点称为 `master` 节点，一个集群 `只能有一个`

* cluster state 存储在每个节点上，master 维护最新版本并同步给其他节点

* master 节点是通过集群中所有节点选举产生的，可以被选举的节点称为 `master-eligible 节点` ，相关配置如下

```
node.master: true
```

上面启动的node1节点就是 master 节点，因为集群就只有它一个

![](https://images.notes.xuepincat.com/elastic/elasticsearch/67.png)

cerebro 中的星星就是表示 master 节点的意思

#### 创建一个索引

API

```
PUT test_index
```

使用 cerebro 自带的 create index

![](https://images.notes.xuepincat.com/elastic/elasticsearch/68.png)

输入索引名称，点击创建按钮，页面会提示对应的信息

![](https://images.notes.xuepincat.com/elastic/elasticsearch/69.png)

接着会到 overview 中查看

![](https://images.notes.xuepincat.com/elastic/elasticsearch/70.png)

索引创建完之后索引的信息会被更新到 cluster state,cluster state 本身是有版本一说，之前是 cluster state 0,现在创建了test_index索引之后就会变成 cluster state 1

#### Coordinating Node

处理请求的节点称为 `coordingating 节点`，比如上面发起一个 `PUT test_index` 创建索引的请求,c处理这个请求的酒店就是 `coordingating 节点` , 该节点为所有节点的默认角色，不能取消

该节点的作用是将请求路由到正确的节点处理，比如创建索引的请求路由到 master 节点去创建，因为创建索引的时候需要修改 cluster state,能够修改 cluster state 的只有 master节点，所以 coordingating 节点就会把创建索引的请求路由到 master 节点去操作

#### Data Node

存储数据的节点称为 `data 节点`，默认节点都是 data 类型，相关配置：

```
node.data: true
```

创建了一个索引后再去创建一些数据，这些数据会存储在一个节点上，能够存储数据的节点称为 data 类型的节点

#### 单点问题

现在集群只有一个节点，如果 node1 节点停止服务，那么整个集群就停止服务

解决的方法就是 `新增节点`

#### 新增一个节点

运行如下命令可以启动一个es节点实例

```
bin/elasticsearch -Ecluster.name=my_cluster -Epath.data=my_cluster_node2 -Enode.name=node2 -Ehttp.port=5300 -d
```

cerebro 刷新后出现下面页面,一个新的 node2 节点新加进来了

![](https://images.notes.xuepincat.com/elastic/elasticsearch/71.png)

## 副本与分片

#### 提高系统可用性

1. 服务可用性

2个节点的情况下，允许其中1个节点停止服务，这个集群还是可以对外提供服务的

2. 数据可用性

数据可用性是指即使一个节点挂掉之后整个数据还是完整的，es是引入副本(Replication) 来解决的，也就是说在每个节点上都有完备的数据，这样即便一个节点挂掉，在另一个节点上依然可以获取到完整的内容

#### 副本

比如test_index的数据在node1上有，在node2上也有，note2 上是 test_index 的副本，这个时候如果 node1 挂了，node2 上依然是获取到 test_index 的所有数据的

#### 增大系统容量

* 如何将数据分布到所有节点上？

比如现在有两个节点，三个节点，十个节点，甚至更多，如果将数据分布在多个节点上，充分利用多个节点的存储资源呢？

es 引入了 `分片(Shard)` 的概念来解决这个问题

* 分片是es支持 PB 级数据的基石

因为 分片存储了部分数据，可以分布于任意节点上；
另外分片数在索引创建时指定且后续不允许再更改，默认为5个；
分片有主分片和副分片之分，以实现数据高可用；
副本分片的数据由主分片同步，可以有多个副本分片，从而提高读取的吞吐量，副本分片数是可以随时动态调整的

#### 分片

创建一个 test_index ,创建的时候指定 shards 数为3，副本数为1，也就是有3个主分片，一个副本，总共有6个分片

API

```
PUT test_index
{
    "setings": {
        "number_of_shards": 3,
        "number_of_replicas": 1
    }
}
```

现在再添加一个节点

```
bin/elasticsearch -Ecluster.name=my_cluster -Epath.data=my_cluster_node3 -Enode.name=node3 -Ehttp.port=5400 -d
```

![](https://images.notes.xuepincat.com/elastic/elasticsearch/72.png)

现在将之前创建的 test_index 删除，如没有就不用删了，测试用的，

![](https://images.notes.xuepincat.com/elastic/elasticsearch/73.png)

然后创建新的 tetst_index,shards 为 3，replicas 数为 1

![](https://images.notes.xuepincat.com/elastic/elasticsearch/74.png)

再回到 overview 中查看

![](https://images.notes.xuepincat.com/elastic/elasticsearch/75.png)

所有的分片都已经分配了，其中实线框框表示主分片，虚线框框表示副本分片

现在已经有了三个节点，test_index 索引有3个分片
1. 现在如果增加节点是否能提高 test_index 的数据容量？因为新加了节点就意味着新加了内存资源，磁盘资源，es是一个分布式系统，是可以水平扩容的

- 不能，因为只有3个分片，已经分布在3台节点上，新增的节点无法利用

2. 一个索引它的副本数是可以有多个的，如果此时增加副本数是否能提高 test_index 读取吞吐量？

- 不能，因为新增的副本也是分布在3个节点上，还是利用了同样的资源，如果用增加吞吐量，还需要新增节点

分片数的设定很重要，需要提前规划好，过小导致后续无法通过增加节点实现水平扩容；过大会导致一个节点上分布过多分片，造成资源浪费，同时会影响查询性能

## 集群状态 Cluster Health

通过如下 api 可以查看集群健康状况，有下面三种

1. green 健康状态，指所有主副分片都正常分配

2. yellow 指所有主分片都正常分配，但是有副本分片未正常分配

3. red 有主分片未分配

- request

```
GET _cluster/health
```

- response

```
{
  "cluster_name": "elasticsearch",
  "status": "yellow",
  "timed_out": false,
  "number_of_nodes": 1,
  "number_of_data_nodes": 1,
  "active_primary_shards": 29,
  "active_shards": 29,
  "relocating_shards": 0,
  "initializing_shards": 0,
  "unassigned_shards": 28,
  "delayed_unassigned_shards": 0,
  "number_of_pending_tasks": 0,
  "number_of_in_flight_fetch": 0,
  "task_max_waiting_in_queue_millis": 0,
  "active_shards_percent_as_number": 50.877192982456144
}
```

我使用 cerebro 链接的是 `http://localhost:9200`

![](https://images.notes.xuepincat.com/elastic/elasticsearch/76.png)

## 故障转移

比如集群由3个节点组成，并且集群状态是 green,如果 node1 所在的机器宕机导致服务终止，其实 node2和node3会定时去ping node1，node2和node3无法响应一段时间后会发起 master 选举，比如会选择 node2 为 master 节点，此时主分片 P0 下线，主分片没有了，集群状态就会变成 red，node2发现主分片P0没有分配之后会把副本分片 R0 变成P0，提升为主分片，此时由于所有主分片都正产分配，集群状态变成 Yellow; node2 为 P0 和 P1 生成新的副本，集群状态变成绿色

比如下图有三个节点，node1 是 master 节点

![](https://images.notes.xuepincat.com/elastic/elasticsearch/77.png)

现在将 node1 关掉,需要先找到 node1

```
ps aux|grep -i elasticsearch|grep 5200|grep node1
```

![](https://images.notes.xuepincat.com/elastic/elasticsearch/78.png)

杀掉他

```
kill 71620
```

刷新一下页面发现连不上了，因为连接的是5200的端口

![](http://images.notes.xuepincat.com/elastic/elasticsearch/79.png)

现在连接 5300，发现node3已经是主节点了,并且分片 0，1，3也已经是正常的状态

![](https://images.notes.xuepincat.com/elastic/elasticsearch/80.png)

## 文档分布式存储

文档最终会存储在分片上，比如发起一个文档创建请求

```
PUT test_index/doc/1
{
    "tiele": "hello, world",
    "desc": "nothing here"
}
```

假设文旦最终存储在分片 P1 上，在 P1 和 副本 R1 上都会有这个文档

那么 Document 1 是如何存储到分片 P1 上的？ 选择 P1 的依据是什么?

有一个文档到分片的映射算法， Document 并不是随便选择 P1 的，为什么要设定这个算法呢？目的是使得文档均匀分布在所有分片上，以充分利用资源，而不会发生所有文档都分布在P0上，P1上面的分配的文档就会P1这台机器资源是完全浪费的，而P0存储了过多的文档，导致它的负载非常高

算法：随机选择或者 round-robin 算法？

随机算法比如有三个分片，随机选择一个分片，或者用 round-robin 轮询算法，这个能实现均匀分布的目的，随机选择和轮询肯定是可以实现均匀分布的目标，这种方法实际是不可取的，是因为需要维护文档到分片的映射关系， 成文巨大，因为文档存进去之后还是需要读取的，比如 Document 存到 P1 了，现在有个请求需要根据id:1获取这个文档，如果是随机存储或者 round-robin 存储的，是无法知道 document1 存储在哪个分片上的，就会出现两个情况，一种是请求来的时候去所有分片上都请求一下document id=1,有的话就返回，这个坏处就是如果有100个，一个读请求就都去请求，很可能100个里面99个都是浪费的，文档只分布在一个分片，这是个巨大的性能问题，另一个就是专门开辟一个存储空间，存储文档到分片的映射关系，当一个请求来就去查下，这个方案就是需要很大的成文来维护文档到分片的映射关系

正确做法是根据文档值实时计算对应的分片

算法：

```
shard = hash(routing) % number_of_primary_shards
```

hash 算法保证可以将数据均匀地分散在分片中

routing 是一个关键参数，默认是文档id,也可以自行指定

number_of_primary_shards 主分片数

该算法与主分片数相关，这也是分片数一旦确定后便不能更改的原因

* 文档创建流程

1. Client 向 node3 发起创建文档请求，node3 作为接收请求的节点；

2. node3 通过 routing 计算该文档应该存储在 shard 1 上，查询 cluster state 后确认主分片 P1 在 node2 上，然后转发创建文档的请求到 node2;

3. node 2 上的 P1 接收并执行创建文档请求后，经同样的请求发送到副本分片R1，同样也是通过 cluster state 确认 R1 在 node1 上；

4. R1 接收并创建文档请求后，通知 P1 成功的结果；

5. P1 接收副本分片结果后，通知 node3 创建成功；

6. node3 返回结果到 Client

* 文档读取流程

1. Client 向 node3 发起获取文档 1 的请求；

2. node3 通过 routing 计算该文档存储在 shard 1 上，查询 cluster state 后获取 shard 1 的主副分片列表，然后以轮询的机制获取一个 shard.比如R1,然后转发读取文档的请求到 node1;

3. R1 接收并执行读取文档请求后，将结果返回node3;

4. node3 返回结果给 Client

* 文档批量创建的流程

1. Client 向 node3 发起批量创建文档的请求(bulk)

2. node3 通过 routing 计算所有文档对应的 shard,然后按照 shard 分配对应执行的操作，同时发送秦秋到设计的主 shard,比如3个主 sahrd 都需要参与；

3. 主 shard 接收并执行请求后，将同样的请求同步到对应的副本 shard

4. 副本 shard 执行结果后返回结果到主 shard,主 shard 再返回 node3

5. node3 整合结果后返回 Client

* 文档批量读取流程

1. Client 向 node3 发起批量获取文档的请求(mget)

2. node3 通过 routing 计算所有文档对应的 shard,然后以轮询的机制获取要参与的 shard,按照 shard 构建mget 请求，同时发送请求到设计的 shard,比如有2个shard需要参与

3. R1,R2返回文档结果

4. node3 返回结果给 Client 

## shard

倒排索引一旦创建，不能更改

优点：

- 不用考虑并发写文件问题，杜绝了锁机制带来的性能问题

- 由于文件不再更改，可以充分利用文件缓存，只需载入一次，质押内存足够，对该文件的读取都会从内存读取，性能高

- 因为倒排索引文件不会变更，就意味着倒排索引的数据是不会变更的，所以很好的针对倒排索引文件生成缓存，而且缓存是不用去维护它的变更性的

- 利于对文件进行压缩存储，节省磁盘和内存存储空间

坏处：

- 需要写入新文档时，必须重新构建倒排索引文件，然后替换老文件后，新文档才能被检索，导致文档实时性差

### 场景

假设现在有个倒排索引文件，用户查询文档的时候都是从倒排索引中查的，现在有一些新文件要写入到倒排索引文件来供用户查询

1. 首先会根据新写入的文档来重新构建倒排索引文件，生成新的倒排索引文件

2. 用户之前是从旧的倒排索引文件中查询的，此时会将用户与旧的倒排索引文件的关联切掉，切换到新的倒排索引文件中来，这样用户就会查询到新写入的文档

这种情况会出现一个问题，就是这种构建过程会导致开销非常大，因为是对所有的倒排索引文件里面的数据进行了重新构建，如果数据量非常大就会导致索引文件非常大，花费的时间也会很多，就会导致文档被用户查询的时长间隔也会非常大，就会导致实时性很差

解决方案：

当有新文档的时候针对新文档直接生成新的倒排索引文件，查询的时候同时查询所有的倒排文件，也就是同时查询旧的倒排索引文件和新的倒排索引文件，然后将倒排索引文件返回的结果汇总计算

这么做的优点是针对新文档构建倒排索引文件速度是很快的，因为文档数不是很多，构建倒排索引开销小，也就意味着能够更快的被用户查询到，通过这种方案就可以提高文档查询的实时性

### 文档搜索实时性

`Lucene` 就是采用的上面的这种方案，它构建的单个倒排索引称为 `segment`, Lucene 里面 segment 是实际上的倒排索引，segment 合在一起称为 `Index`, 与 ES 中的 Index 概念不同,ES 中的 Index 是逻辑上Document的集合，Lucene的Index是 Segment 的集合，ES 中的 `Shard` 对应一个 `Lucene Index`

Lucene 会有一个专门的文件来记录所有的 `segment` 信息，称为 `commit point`，也就是 Lucene 有个叫 commit point 的文件，它会记录当前的 Lucene Index 中有多少个segment,查询的时候就可以知道查询多少segment

#### refresh

segment 写入磁盘的过程依然很耗时，如果想进一步加快文档的实时性可以借助文件系统缓存的特性，现将 segment 在缓存中创建并开放查询来进一步提升实时性，也就是在内存中将 segment 创建好并开放出来，不必等 segment 写入磁盘，通过这种操作就可以提升文档被查询的实时性，这个过程在es中被称为 `refresh`

es 在 refresh 之前文档辉县存在在一个 buffer 中,一个缓冲队列，refresh 时将 buffer 中的所有文档清空并生成 segment

es 默认 `每1秒` 执行一次 refresh, 因此文档的实时性被提高到 1秒，这也是es被称为近实时(Near Real Time) 的原因

#### translog

* translog 解决什么问题？

如果内存中的 segment 还没有接入磁盘前，es所在的机器发生了宕机，那么在内存中的文档就无法恢复了

* 如何解决这个问题？

- es 引入了 `translog` 机制,通过这个机制可以减少当这种情况出现的时候对文档造成的损失。写入文档到 buffer 时，同时将该操作写入 translog,也就是说 translog 也有一份记录，增删改查在translog中也会有一份记录，translog 会有自己备份的机制，有自己的写入控制，index buffer 会通过 refresh 操作生成 segment in memory, translog 会通过 fsync 操作直接将内容落盘到磁盘上，生成 Translog File

只要 translog 能够保证自己的一些文件能够尽快落到磁盘里，就可以解决放生宕机的时候 segment in memory 里面文档丢失的风险

- translog 文件会即时写入磁盘(fsyns),6.x 默认每个请求都会落盘，可以修改为每5秒写一次，这样风险便是丢失5秒内的数据，相关配置为 `index.translog.*`

- es 启动时会检查 translog 文件，并从中恢复

#### flush

flush 负责将内存中的 segment 写入磁盘

- 将 translog 写入磁盘

- 将 index buffer 清空，其中的文件生成一个新的segment,相当于一个 refresh 操作

- 更新 commit point 并写入磁盘

- 执行 fsync 操作，将内存中的 segment 写入磁盘

- 删除旧的 translog 文件

#### refresh 发生时机

- 间隔时间到达时，通过 `index.setting.refresh_interal` 来设定，默认是1秒

- index.buffer 占满时，其大小通过 `indices.memory.index_buffer_size` 设置，默认为 jvn heap 的10%，所有 shard 共享

- flush 发生时也会发生 refresh

#### flush 发生时机

- 间隔到达时，默认是30分钟，5.x之前可以通过 `index.translog.flush_threshold_period` 修改，之后无法修改

- translog 占满时，其大小可以通过 `index.translog.flush_threshold_size` 控制，默认是512mb,每个 index 都有自己的 translog

#### 删除与更新文档

* segment 一旦生成就不能更改，那么如果要删除文档该如何操作呢？

- Lucene 专门维护一个 .del 的文件，记录所有已经删除的文档，注意 .del 上记录的是文档在 Lucene 内部的id

- 在查询结果返回前会过滤掉 .del 中的所有文旦

* 更新文档如何操作？

- 首先删除文档，然后再创建新文档

#### Segment Merging

* 随着 segment 的增多，由于一次查询的 segment 数增多，查询速度会变慢

每一次 refresh 都会生成一个 segment,每次flush ，segment都会写入到磁盘里，es每秒钟都会refresh一次，也就是每秒钟都会生成一个segment,一个小时就会生成 3600个 segment,segment 会越来越多，就会导致每次查询的时候实际上是对所有的segment做查询，随着 segment 数增多，查询性能也会变慢，查询速度也会拖慢

* es 会定时在后台进行 segment merge 操作，减少 segment 的数量

es 会定时在后台进行 segment merge 操作，也就是会将小的segment 合并为大的 segment,从而减少 segment 数量，同时也会将 .del 文件删除掉，这样就能优化 segment 的读取性能

* 通过 force_merge api 可以手动强制做segment merge 的操作s

#### 综合

 比如 ES Index 下有三个 Shard,Shard0,Shard1,Shard3,那么每个Shard就是一个 Lucene Index,一个 Lucene Index 是由一堆 segment 组成的，比如有 segment0,segment1,segment2...，会有一个 commit point 文件来维护这些 segment0,segment1,segment2...,同时还会有 .del 文件来记录已经删除的文档


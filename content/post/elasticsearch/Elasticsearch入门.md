+++
title = 'Elasticsearch入门'
date = 2020-05-18T23:00:38+08:00
draft = true
categories = [ "ElasticSearch" ]
tags = [ "es" ]
+++

# 基本概念

* 文档 Document

  - 用户存储在 ES 中的数据文档，是 ES 中存储的最小数据单元，相当于 MySQL 表中的一行数据

* 索引 Index

  - 由具有相同字段的文档列表组成，是一个文档的集合，在 MySQL 中对应的是一个 Table

> 可能之前会将 Index 对比成数据库中的 Database，原因是 Index 下有不同的 Type ，即不同的类型，但在 6.0 之后官方已经不允许在一个 Index 下创建多个 Type 了，也就是说一个 Index 只有一个 Type，并且在未来的大版本中会将 Type 的概念去掉，所以将 Index 类比成 Database 已经不太合适了，所以这里就直接类比成 MySQL 中的 Table 

* 节点 Node

  - 一个 Elasticsearch 的运行实例，是集群的构成单元，也就是集群是由多个节点构成的，每个节点就是 ES 的一个运行实例

* 集群 Cluster

  - 由一个或多个节点组成，对外提供服务

# Document

* Json Object，Document 在 ES 中实际就是一个 `Json Object` , 一个 `Json Object` 是由 `字段 (Field)` 组成，每个字段有不同的类型

  - 字符串类型: text, keyword，区别是 text 是分词的，keyword 是不分词的
  - 数值型: long, integer, short, byte, double, float, half_float, scaled_float。其中 half_float, scaled_float 是一些比较节省空间的类型。比如有需要保存男生女生的字段，这些字段都是固定的，0代表女生，1代表男生，就可以使用 byte 类型，这样可以大大节省磁盘空间
  - 布尔: boolean
  - 日期: date
  - 二进制: binary
  - 范围类型: integer_range, float_range, long_range, double_range, date_range

* 每个文档都有一个唯一的id标识，类似 MySQL 中的主键 (Primary Key)

  - 自行指定
  - 自动生成

比如有一条 nginx log，格式如下：

```
93.180.71.3 -- [17/May/2015:08:05:32 +0000 "GET /downloads/product_1 HTTP/1.1" 304 0 "-" "Debian APT-HTTP/1.3 (0.8.16~exp12ubuntu10.21)"]
```

document in es：

```
{
    "remote_ip": "93.180.71.3", 
    "user_name": "-",
    "@timestamp": "2015-05-17T08:05:32.000Z",
    "request_action": "GET",
    "request": "/downloads/product_1",
    "http_version": "1.1",
    "response": "304",
    "bytes": "0",
    "referrer": "-",
    "agent": "Debian APT-HTTP/1.3 (0.8.16~exp12ubuntu10.21)"
}
```

格式：`field_name: field_value` 

# Document MetaData

* 元数据，用于标注文档的相关的原始信息

  - _index: 文档所在的索引名；
  - _type: 文档所在的类型名；
  - _id: 文档唯一id；
  - _uid: 组合id, 由 _type 和 _id 组成 (6.x _type 不再起作用，同 _id 一样)；
  - _source: 文档的原始 Json 数据，可以从这里获取每个字段的内容；
  - _all: 整合所有字段内容到该字段，可以对Document的所有内容做查询，官方现在已经不推荐使用_all了，因为它是针对所有字段进行分词，所以会比较占磁盘空间，并且它的查询效果也并不是很好，在新的版本中已经默认禁用了

# Index

* 索引中存储具有相同结构的文档(Document)

> Index 在数据库中类比 Table，Table都是有自己的表结构定义的，ES 中每个Index也有自己的 Schema 定义。

  - 每个索引也都有自己的 `mapping` 定义，用于定义字段名和类型

* 一个集群可以有多个索引，比如：nginx 日志存储的时候可以按照日期每天生成一个索引来存储，方便维护

  - nginx-log-2020-05-16
  - nginx-log-2020-05-17
  - nginx-log-2020-05-18

# REST API

* Elasticsearch 集群对外提供了一些 RESTful API

  - REST 全称为 Representational State Transfer（表现层状态转移），理解为对资源做一些操作后，资源的状态会发生变化；
  - URI 指定资源，如 Index 是一个资源，可以对它进行创建，删除等, Document 也是资源，也可以进行创建，更新，删除等
  - Http Method 指明资源操作类型，如 GET、POST、PUT、DELETE 等

* ES 中有两种常用交互方式

  - Curl 命令行
  - Kibana

# 索引 API (index api)

* ES 有专门的 Index API， 用于创建、更新、删除索引配置等

## 创建索引 API

* request

```
# create index
PUT /test_index
```

test_index 是索引名

* response

```
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "index": "test_index"
}
```

![](https://images.notes.einscat.com/elastic/elasticsearch/4.png)

## 查看现有索引

* reqeust

```
GET _cat/indices
```

* response

```
yellow open test_index x-1Q_hp-TgOqsiIFs9SKYg 5 1 0 0 1.1kb 1.1kb
```

![](https://images.notes.einscat.com/elastic/elasticsearch/5.png)

## 删除索引

```
DELETE /test_index
```

![](https://images.notes.einscat.com/elastic/elasticsearch/6.png)

# 文档 Document API

es 有专门的 Document API

## 创建文档

**指定 id 创建文档**

创建文档时，如果索引不存在，es会自动创建对应的 index 和 type

```
# create document
PUT test_index/doc/1
{
  "username": "alfred",
  "age": 1
}
```

```
{
  "_index": "test_index",
  "_type": "doc",
  "_id": "1",
  "_version": 1,
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 0,
  "_primary_term": 1
}
```

![](https://images.notes.xuepincat.com/elastic/elasticsearch/7.png)

**不指定id创建文档API**

这种情况 HTTP Method 变成了 POST，需要注意下

```
# create document without id
POST test_index/doc
{
  "username": "tom",
  "age": 20
}
```

```
{
  "_index": "test_index",
  "_type": "doc",
  "_id": "KBb8PHIBjbQNbnLLlH-i",
  "_version": 1,
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 0,
  "_primary_term": 1
}
```

![](https://images.notes.xuepincat.com/elastic/elasticsearch/8.png)

## 查询文档

**指定要查询的文档id**

```
# query document
GET /test_index/doc/1
```

```
{
  "_index": "test_index",
  "_type": "doc",
  "_id": "1",
  "_version": 1,
  "found": true,
  "_source": {
    "username": "alfred",
    "age": 1
  }
}
```

  _source 存储了文档的完整原始数据

![](https://images.notes.xuepincat.com/elastic/elasticsearch/9.png)

**查找不存在的文档**

```
GET /test_index/doc/2
```

```
{
  "_index": "test_index",
  "_type": "doc",
  "_id": "2",
  "found": false
}
```

![](https://images.notes.xuepincat.com/elastic/elasticsearch/10.png)

**搜索所有文档，使用 `_search`**

无请求体:

```
# query
GET /test_index/doc/_search
```

```
{
  "took": 42, # 表示查询耗时，单位ms
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 2, # 符合条件的总文档数
    "max_score": 1,
    "hits": [ # 返回的文档详情数据数组，默认前10个文档
      {
        "_index": "test_index", # 索引名
        "_type": "doc",
        "_id": "KBb8PHIBjbQNbnLLlH-i", # 文档id
        "_score": 1, # 文档得分
        "_source": { # 文档详情
          "username": "tom",
          "age": 20
        }
      },
      {
        "_index": "test_index",
        "_type": "doc",
        "_id": "1",
        "_score": 1,
        "_source": {
          "username": "alfred",
          "age": 1
        }
      }
    ]
  }
}
```

![](https://images.notes.xuepincat.com/elastic/elasticsearch/11.png)

有请求体:

```
# query
GET test_index/doc/_search
{
  "query": {
    "term": {
      "_id": "1"
    }
  }
}
```

```
{
  "took": 10,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 1,
    "hits": [
      {
        "_index": "test_index",
        "_type": "doc",
        "_id": "1",
        "_score": 1,
        "_source": {
          "username": "alfred",
          "age": 1
        }
      }
    ]
  }
}
```

![](https://images.notes.xuepincat.com/elastic/elasticsearch/12.png)

查询语句，JSON格式，放在 `http body` 中发送到 es

## 批量创建文档

es 允许一次创建多个文档，从而减少网路传输开销，提升写入速率，

endpoint: _bulk
http method: POST
action_type:
  index - 创建，当文档存在，则会将文档覆盖掉
  update - 更新
  create - 创建，当文档已经存在的时候就会报错
  delete - 删除文档

**首先创建一个id为3的文档**

```
PUT /test_index/doc/3
{
  "username": "alfred",
  "age": 10
}
```

```
{
  "_index": "test_index",
  "_type": "doc",
  "_id": "3",
  "_version": 1,
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 0,
  "_primary_term": 1
}
```

**bulk批量创建**

```
# bulk api
POST _bulk
{"index":{"_index":"test_index","_type":"doc","_id":"3"}}
{"username":"alfred","age":20}
{"delete":{"_index":"test_index","_type":"doc","_id":1}}
{"update":{"_id":"2","_index":"test_index","_type":"doc"}}
{"doc":{"age":"20"}}
```

```
{
  "took": 91,
  "errors": true,
  "items": [
    {
      "index": {
        "_index": "test_index",
        "_type": "doc",
        "_id": "3",
        "_version": 2,
        "result": "updated",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 1,
        "_primary_term": 1,
        "status": 200
      }
    },
    {
      "delete": {
        "_index": "test_index",
        "_type": "doc",
        "_id": "1",
        "_version": 2,
        "result": "deleted",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 2,
        "_primary_term": 1,
        "status": 200
      }
    },
    {
      "update": {
        "_index": "test_index",
        "_type": "doc",
        "_id": "2",
        "status": 404,
        "error": {
          "type": "document_missing_exception",
          "reason": "[doc][2]: document missing",
          "index_uuid": "QeEur8ddQrSL4_H0b4pcIA",
          "shard": "2",
          "index": "test_index"
        }
      }
    }
  ]
}
```

errors 为 true 表示出错了，因为不存在id为2的文档

## 批量查询

es 允许一次查询多个文档,可以一次性指定多个id

**mget**

endpoint: _mget

```
# multi_get
GET /_mget
{
  "docs": [
    {
      "_index": "test_index",
      "_type": "doc",
      "_id": "3"
    },
    {
      "_index": "test_index",
      "_type": "doc",
      "_id": "2"
    }
  ]
}
```

```
{
  "docs": [
    {
      "_index": "test_index",
      "_type": "doc",
      "_id": "3",
      "_version": 2,
      "found": true,
      "_source": {
        "username": "alfred",
        "age": 20
      }
    },
    {
      "_index": "test_index",
      "_type": "doc",
      "_id": "2",
      "found": false
    }
  ]
}
```
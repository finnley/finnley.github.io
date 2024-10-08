+++
title = 'Search运行机制'
date = 2020-06-17T00:35:23+08:00
draft = true
categories = [ "ElasticSearch" ]
tags = [ "es" ]
+++

## Search 运行机制

* Search 执行的时候实际分两个步骤运行的

- Query 阶段

- Fetch 阶段

* Query-Then-Fetch

### Query 阶段

用户向集群发起查询请求 `GET test_index/_search`,某一个节点比如node3 在接收到用户的search请求后，会先进行 Query 阶段（此时是Node3作为 Coordinating Node 角色在服务）

Node3会在6个主副分片中随机选择3个分片，发送 search request

被选中的3个分片会分别执行查询并排序，返回 from+size 个文档Id和排序值，比如返回第10-19个文档，实际在执行的时候每个分片都会返回es 10+10=20个文档，node3一共拿到 3*20=60个文档，然后在这60个文档里面通过排序之后再去拿实际的第9到19个文档，这个就是es的分片机决定了node3没法知道真正的前20个文档在哪个分片，有可能20个文档在P0,P1上，和P2没有关系，也可能20个文档均匀的分布在P0,P1,P2三个分片上，这种情况就需要每个分片把前20个分片取出来做一个整合，在整合之后就可以算出真正的前20个文档了

Node3 整合3个分片返回的 front+size 个文档Id,根据排序值排序后选取from到from+size的文档Id

### Fetch 阶段

* node3 根据 Query 阶段获取的文档Id列表去对应的shard上获取文档情况数据

node3 向相关分片发送 multi_get 请求

3个分片返回文档详细数据

node3 拼接返回的结果并返回给客户

## 相关性算分

* 相关性算分在 shard 与 shard 间是相互独立的，也就以为这同一个 Term 的 IDF等一些值在不同的 shard 上是不同的。文档的相关想算分和它所处的 shard 相关

* 在文档数量不多时，会导致相关性算分严重不准的情况发生

比如 倒排索引 这个 term 在shard0上有10个文档，这10个文档包括倒排索引关键词，在shard1只有一个文档包含倒排索引这个关键词，在分片2上有5个文档包含倒排索引关键词，由于在不同的shard上有不同的 document refrequence(文档频率)，也就意味着 term 在 不同的分片上进行算分时，计算得到的结果是完全不同的，也就会导致算分时不准的，可能分片1上的相关性得分高于分片0上的得分

下面比如添加三个文档

```


DELETE test_search_relevance

POST test_search_relevance/doc
{
  "name": "hello"
}

POST test_search_relevance/doc
{
  "name": "hello,world"
}

POST test_search_relevance/doc
{
  "name": "hello,world! a beautiful world"
}
```

接着获取下

```
GET test_search_relevance/_search
```

再查看下索引的配置

```
GET test_search_relevance
```

获取 name 是 hello 的 文档

```
GET test_search_relevance/_search
{
  "query": {
    "match": {
      "name": "hello"
    }
  }
}
```

返回了三个文档，但是我想要的只有 hello 的文档

可以添加 `explain` 查看下具体情况

```
GET test_search_relevance/_search
{
  "explain": true,
  "query": {
    "match": {
      "name": "hello"
    }
  }
}
```

* 解决方案

1. 是设置分片数为1个，从根本上排除问题，在文档数量不多的时候可以考虑该方案比如百万到千万级别的文档数量

2. 使用 DFS Query-then-Fetch 查询方式

DFS Query-then-Fetch 是在拿到所有文档后再重新完整的计算一次相关性算分，耗费更多的cpu和内存，执行性能也比较地下，一般已建议使用。

使用方式如下：

```
GET test_search_relevance/_search?search_type=dfs_query_then_fetch
{
  "query": {
    "match": {
      "name": "hello"
    }
  }
}
```

## sorting-doc-values-field-data

### 排序

* es 默认是采用相关性算分从高到低排序的，用户可以通过设定 sorting 参数来自行设定排序规则

比如按照出声日期排序排列

```
GET test_search_index/_search
{
  "sort": {
    "birth": "desc"
  }
}
```

也可以执行多个排序条件，比如按照多个字段排序，分别按照出声日期倒排，得分倒排，文党内部ID倒排

```
GET test_search_index/_search
{
  "sort": [ # 关键词
    {
      "birth": "desc"
    },
    {
      "_score": "desc" # _score 指的是相关性得分
    },
    {
      "_doc": "desc" # _doc 指的是文档内部Id,和索引相关（分片内唯一）
    }
  ]
}
```

1. 根据出声日期排序

```
# score is null here
GET test_search_index/_search
{
  "query": {
    "match": {
      "username": "alfred"
    }
  },
  "sort": [
    {
      "birth": {
        "order": "desc"
      }
    }
  ]
}
```

- response

```
{
  "took": 27,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 3,
    "max_score": null,
    "hits": [
      {
        "_index": "test_search_index",
        "_type": "doc",
        "_id": "1",
        "_score": null,
        "_source": {
          "username": "alfred way",
          "job": "java engineer",
          "age": 18,
          "birth": "1990-01-02",
          "isMarried": false
        },
        "sort": [
          631238400000
        ]
      },
      {
        "_index": "test_search_index",
        "_type": "doc",
        "_id": "4",
        "_score": null,
        "_source": {
          "username": "alfred junior way",
          "job": "ruby engineer",
          "age": 23,
          "birth": "1989-08-07",
          "isMarried": false
        },
        "sort": [
          618451200000
        ]
      },
      {
        "_index": "test_search_index",
        "_type": "doc",
        "_id": "2",
        "_score": null,
        "_source": {
          "username": "alfred",
          "job": "java senior engineer and java specialist",
          "age": 28,
          "birth": "1980-05-07",
          "isMarried": true
        },
        "sort": [
          326505600000
        ]
      }
    ]
  }
}
```

上面返回结果中 score 是 null，如果按照日期排序，算分就毫无意义，所以es就没有自动算分

如果需要算分可以使用下面的方式

```
GET test_search_index/_search
{
  "query": {
    "match": {
      "username": "alfred"
    }
  },
  "sort": [
    {
      "birth": {
        "order": "desc"
      }
    },
    {
      "_score": "desc"
    },
    {
      "_doc": "desc"
    }
  ]
}
```

- response

```
{
  "took": 4,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 3,
    "max_score": null,
    "hits": [
      {
        "_index": "test_search_index",
        "_type": "doc",
        "_id": "1",
        "_score": 0.33698124,
        "_source": {
          "username": "alfred way",
          "job": "java engineer",
          "age": 18,
          "birth": "1990-01-02",
          "isMarried": false
        },
        "sort": [
          631238400000,
          0.33698124,
          0
        ]
      },
      {
        "_index": "test_search_index",
        "_type": "doc",
        "_id": "4",
        "_score": 0.27601978,
        "_source": {
          "username": "alfred junior way",
          "job": "ruby engineer",
          "age": 23,
          "birth": "1989-08-07",
          "isMarried": false
        },
        "sort": [
          618451200000,
          0.27601978,
          3
        ]
      },
      {
        "_index": "test_search_index",
        "_type": "doc",
        "_id": "2",
        "_score": 0.43250346,
        "_source": {
          "username": "alfred",
          "job": "java senior engineer and java specialist",
          "age": 28,
          "birth": "1980-05-07",
          "isMarried": true
        },
        "sort": [
          326505600000,
          0.43250346,
          1
        ]
      }
    ]
  }
}
```

### 字符串排序徐

按照字符串排序比较特殊，因为es有text和keyword两种类型，针对text类型排序会报错

```
GET test_search_index/_search
{
  "sort": {
    "username": "desc"
  }
}
```

- response

![](https://images.notes.xuepincat.com/elastic/elasticsearch/81.png)

错误原因是：Fielddata is disabled on text fields by default. Set fielddata=true on [username] in order to load fielddata in memory by uninverting the inverted index

* 针对keyword类型排序，可以返回预期结果

通过 mapping 可以看到username子字段是keyword类型，可以按照 keyword 进行排序

```
GET test_search_index/_search
{
  "sort": {
    "username.keyword": "desc"
  }
}
```

- response

```
{
  "took": 0,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 4,
    "max_score": null,
    "hits": [
      {
        "_index": "test_search_index",
        "_type": "doc",
        "_id": "3",
        "_score": null,
        "_source": {
          "username": "lee",
          "job": "java and ruby engineer",
          "age": 22,
          "birth": "1985-08-07",
          "isMarried": false
        },
        "sort": [
          "lee"
        ]
      },
      {
        "_index": "test_search_index",
        "_type": "doc",
        "_id": "1",
        "_score": null,
        "_source": {
          "username": "alfred way",
          "job": "java engineer",
          "age": 18,
          "birth": "1990-01-02",
          "isMarried": false
        },
        "sort": [
          "alfred way"
        ]
      },
      {
        "_index": "test_search_index",
        "_type": "doc",
        "_id": "4",
        "_score": null,
        "_source": {
          "username": "alfred junior way",
          "job": "ruby engineer",
          "age": 23,
          "birth": "1989-08-07",
          "isMarried": false
        },
        "sort": [
          "alfred junior way"
        ]
      },
      {
        "_index": "test_search_index",
        "_type": "doc",
        "_id": "2",
        "_score": null,
        "_source": {
          "username": "alfred",
          "job": "java senior engineer and java specialist",
          "age": 28,
          "birth": "1980-05-07",
          "isMarried": true
        },
        "sort": [
          "alfred"
        ]
      }
    ]
  }
}
```

* 如何针对text类型字段排序

排序的过程实质是对字段原始内容排序的过程，这个过程中 `倒排索引无法发挥作用`，需要用到 `正排索引`，也就是通过文档Id和字段可以快速得到字段原始内容,然后拿原始内容排序

es提供了2中实现方式：

1. fielddata 默认禁用

2. doc values 默认启用，除了text类型

|   文档ID  |   字段值    |
|   :----:  |   :----:   |
|   1   |   100    |
|   2   |   89 |
|   3   |   129    |     

* Fielddata vs DocValues

|   对比  |   Fielddata    |  DocValues    |
|   :----:  |   :----:   |  :----:   |
|   创建时机   |   搜索时即时创建    |  索引时创建，与倒排索引创建时机一致    |
|   创建位置   |   JVM Heap | 磁盘 |
|   优点   |   不会占用额外的磁盘资源    |    不会占用 Heap 内存    | 
|   缺点   |   文档过多时，即时创建会花过多时间，占用过多 Heap 内存    |    减慢索引的速度，占用额外的磁盘资源    |    

### Fielddata

Fielddata 默认是关闭的，可以通过下面 api 开启 `"fielddata": true`

开启后字符串是按照分词后的 term 排序，往往结果很难复合预期

一般是在对分词做聚合分析的时候开启

```
PUT test_search_index/_mapping/doc
{
  "properties": {
    "username": {
      "type": "text",
      "fielddata": true
    }
  }
}
```

打开之前执行查询,发现报错了

```
GET test_search_index/_search
{
  "sort": [
    {
      "username": "desc"
    }
  ]
}
```

然后通过下面的配置打开

![](https://images.notes.xuepincat.com/elastic/elasticsearch/82.png)

接着再去执行查询就可以查询了

如果再次关闭后去查询就会再次报错

```
PUT test_search_index/_mapping/doc
{
  "properties": {
    "username": {
      "type": "text",
      "fielddata": false
    }
  }
}
```

fielddata 只能作用于text类型，如果针对其他类型会直接报错

![](https://images.notes.xuepincat.com/elastic/elasticsearch/83.png)

### Doc Values

Doc Values 默认是启用的，可以在创建索引的时候关闭

如果后面要再开启 doc values,需要做reindex操作, Doc Values 相对fielddata 是一个较重的操作，所以最好在一开始设计数据模型的时候就像这些考虑进去，要不然后面需要使用的时候要做reindex操作

当我们明确知道我们不需要按照这个字段来排序，不需要使用该字段做聚合分析，就可以将该字段关闭，这样可以加快索引速度，减少磁盘空间占用

```
PUT test_doc_values1
{
  "mappings": {
    "doc": {
      "properties": {
        "username": {
          "type": "keyword",
          "doc_values": false
        }
      }
    }
  }
}
```

### docvalue_fields

可以通过该字段获取 fielddata 或者 doc values 中存储的内容

```
GET test_search_index/_search
{
  "docvalue_fields": [
    "username",
    "username.keyword",
    "age"
  ]
}
```

```
# doc values
DELETE test_doc_values

PUT test_doc_values

GET test_doc_values

PUT test_doc_values/_mapping/doc
{
  "properties": {
    "username": {
      "type": "keyword",
      "doc_values": false
    },
    "hobby": {
      "type": "keyword"
    }
  }
}

PUT test_doc_values/doc/1
{
  "username": "alfred",
  "hobby": "basketball"
}

GET test_doc_values/_search
{
  "sort": "username"
}
```

执行排序的时候出错了

![](https://images.notes.xuepincat.com/elastic/elasticsearch/84.png)

但是hobby确是可以排序的

```
GET test_doc_values/_search
{
  "sort": "hobby"
}
```

![](https://images.notes.xuepincat.com/elastic/elasticsearch/85.png)

后续如果想把 doc values 打开

```
PUT test_doc_values/_mapping/doc
{
  "properties": {
    "username": {
      "type": "keyword",
      "doc_values": true
    },
    "hobby": {
      "type": "keyword"
    }
  }
}
```

设置成true发现不行,报错了,其实es是不允许后续再把 doc values 打开的

![](https://images.notes.xuepincat.com/elastic/elasticsearch/86.png)

如果要重新打开，需要再重建个索引，然后将原先的索引reindex到新的索引中


下面将 username 的 fielddata 关闭

```
# can be used to get original field value for not stored field
PUT test_search_index/_mapping/doc
{
  "properties": {
    "username": {
      "type": "text",
      "fielddata": false
    }
  }
}

GET test_search_index/_search
{
  "docvalue_fields": [
    "username",
    "username.keyword",
    "age"
  ]
}
```

然后执行上面的es查询会发现报错

![](https://images.notes.xuepincat.com/elastic/elasticsearch/87.png)

但是执行下面es查询时确是正常的,将 username 去掉

```
GET test_search_index/_search
{
  "docvalue_fields": [
    "username.keyword",
    "age"
  ]
}
```

如果将 fielddata 打开，然后再次去执行下es,可以正常访问

```
PUT test_search_index/_mapping/doc
{
  "properties": {
    "username": {
      "type": "text",
      "fielddata": true
    }
  }
}

GET test_search_index/_search
{
  "docvalue_fields": [
    "username",
    "username.keyword",
    "age"
  ]
}
```

## 分页与遍历

es 提供了3中方式来解决分页与遍历问题

1. from/size

2. scroll

3. search_after

### from-size

最常用的分页方案

from: 指明开始位置，从哪个位置开始获取文档

size: 获取总数，从开始的位置指明获取多少个文档

下面表示获取第二个文档，从第一个位置开始，也就是默认排序的情况下取的是第一个和第二个文档，基数从0开始的

```
GET test_search_index/_search
{
  "from": 1,
  "size": 2
}
```

深度分页是一个景点问题：在数据分片存储的情况下如果获取前1000个文档？

from=990,size=10, 即获取从990-1000的文档时，会在每个分片行先获取1000个文档，然后再由Coordinationg Node聚合所有分片的结果后再排序选取前1000个文档

比如es有5个分片,p0, p1, p2, p3, p4,文档存储在5个分片上，es获取总数据中的1000个文档，es会从5个分片上都获取前1000个文档，从p0取前1000个文档， p1前1000个文档，p2前1000个文档，p3前1000个文档，p4前1000个文档，然后整合这5000个文档排序后返回990-1000的文档

这是因为数据分散存储在5个分片上，es的Coordinating Node 是没有一个全局的概念的，它没有办法直接知道整体的数据的前1000个是哪些，它能做的就是从每个分片上取1000个然后排序下，再拿1000个文档就是前1000个文档

这种方案存在个问题，就是页数越深，处理文档阅多，占用内存阅多，耗时越长。尽量避免深度分页，es通过 `index.max_result_window` 限定对多到 10000 条数据

一般分页都会计算一个总页数 total_page

总页数 = (文档总数 + page_size - 1) / page_size

比如 total=4,page_size = 2

total_page = (total+page_size-1)/page_size = (4+2-1)/2=2.5
取整为2

下面就是每页两个数据

第一页

```
# pagination 
GET test_search_index/_search
{
  "from": 0,
  "size": 2
}
```

第二页

```
GET test_search_index/_search
{
  "from": 2,
  "size": 2
}
```

第三页

```
GET test_search_index/_search
{
  "from": 4,
  "size": 2
}
```

如果是深度分页，将from设置为10000,查询结果会直接报错

```
GET test_search_index/_search
{
  "from": 10000,
  "size": 2
}
```

![](https://images.notes.xuepincat.com/elastic/elasticsearch/88.png)

但是如果改成9998就可以

![](https://images.notes.xuepincat.com/elastic/elasticsearch/89.png)

google 搜索引擎也是不允许访问过多的页数，所以限制了页数

### scroll

遍历文档集的api,以快照的方式来避免深度分页的问题

不能用来做实时搜索，因为数据不是实时的，因为建立快照的过程是需要花费时间的，快照是创建快照时的数据情况，后续进来的文档是无法在这次快照中查询到的，所以数据不是实时的

尽量不要使用复杂的 sort 条件，使用_doc 最高效

使用稍显复杂

step 1

需要发起1个 scroll search ，es 在收到该请求后悔根据查询条件创建文档ID合集的快照

- request

```
GET test_search_index/_search?scroll=5m
{
  "size": 1
}
```

scroll=5m: 该 scroll 快照有效时间

size: 每次 scroll 返回的文档数

- response

```
{
  "_scroll_id": "..."
}
```

_scroll_id: 后续调用 scroll api 时的参数

step 2

调用 scroll search 的api,获取文档集合，不断迭代调用知道返回 hits.hits 数组为空时停止

- request

```
POST _search/scroll
{
  "scroll": "5m",
  "scroll_id": "..."
}
```

scroll: 有效时间

scroll_id 上一步返回的 id

- response

```
{
  "_scroll_id": "..."
}
```

_scroll_id: 下一次条用时使用


第一次查询

```
# scroll
GET test_search_index/_search?scroll=5m
{
  "size": 1
}
```

![](https://images.notes.xuepincat.com/elastic/elasticsearch/90.png)

复制返回的 scroll_id

```
POST _search/scroll
{
  "scroll": "5m",
  "scroll_id": "DXF1ZXJ5QW5kRmV0Y2gBAAAAAAABARYWd29qdEd3MUhUM2loTDBQODhmd1Y3dw=="
}
```

![](https://images.notes.xuepincat.com/elastic/elasticsearch/91.png)

再复制下返回的 scroll_id

```
POST _search/scroll
{
  "scroll": "5m",
  "scroll_id": "DXF1ZXJ5QW5kRmV0Y2gBAAAAAAABARYWd29qdEd3MUhUM2loTDBQODhmd1Y3dw=="
}
```

![](http://images.notes.xuepincat.com/elastic/elasticsearch/92.png)

再复制下返回的 scroll_id

```
POST _search/scroll
{
  "scroll": "5m",
  "scroll_id": "DXF1ZXJ5QW5kRmV0Y2gBAAAAAAABARYWd29qdEd3MUhUM2loTDBQODhmd1Y3dw=="
}
```

![](http://images.notes.xuepincat.com/elastic/elasticsearch/93.png) 

知道 hits.hits 数组是空

scroll 是对文档做一个快照，创建快照的时候如果有新文档进来，后面进来的新文档通过 scroll 是取不到的

```
GET test_search_index/_search
```
 查询索引目前是有4个文档

 现在重新创建一个新的 scroll

 ```
GET test_search_index/_search?scroll=5m
{
  "size": 1
}
 ```

 保存下返回的 scroll_id,并创建个新的文档

 ```
# DXF1ZXJ5QW5kRmV0Y2gBAAAAAAABAVMWd29qdEd3MUhUM2loTDBQODhmd1Y3dw==

# new doc can not be searched
PUT test_search_index/doc/10
{
  "username": "doc10"
}
 ```

 POST _search/scroll
{
  "scroll": "5m",
  "scroll_id": "DXF1ZXJ5QW5kRmV0Y2gBAAAAAAABAVMWd29qdEd3MUhUM2loTDBQODhmd1Y3dw=="
}

 ![](https://images.notes.xuepincat.com/elastic/elasticsearch/94.png)

 会发现 total 还是4，也就意味着刚才新进来的文档是没有在创建 scroll 的快照里面

 现在再重新创建个快照

 ```
GET test_search_index/_search?scroll=5m
{
  "size": 1
}
```

保存下返回的 scroll_id,并创建个新的文档

将上面创建的文档10删掉

```
DELETE test_search_index/doc/10
```

删除之后total变成4

现在执行下快照,会看到 total 是5，虽然将文档删掉了

```
POST _search/scroll
{
  "scroll": "5m",
  "scroll_id": "DXF1ZXJ5QW5kRmV0Y2gBAAAAAAABAXMWd29qdEd3MUhUM2loTDBQODhmd1Y3dw=="
}
```

![](https://images.notes.xuepincat.com/elastic/elasticsearch/95.png)

过多的 scroll 调用会占用大量内存，可以通过 clear api 删除过多的 scroll 快照

```
DELETE /_search/scroll
{
  "scroll_id": [
    "scroll_id_1",
    "scroll_id_2",
    "..."
  ]
}
```

或者

```
DELETE /_search/scroll/_all
```

```
DELETE /_search/scroll/_all

POST _search/scroll
{
  "scroll": "5m",
  "scroll_id": "DXF1ZXJ5QW5kRmV0Y2gBAAAAAAABAXMWd29qdEd3MUhUM2loTDBQODhmd1Y3dw=="
}
```

比如我将 scroll 全部删掉，然后再去执行之前的快照,会发现报错了

![](https://images.notes.xuepincat.com/elastic/elasticsearch/96.png)

### search after

search after 避免了深度分页的性能问题，提供实时的下一页文档获取功能

缺点是不能使用 from 参数，即不能指定页数

只能下一页，不能上一页

使用简单

step 1 

正常搜索，但是要指定 sort 值，并保证值唯一，因为 search after 获取下一页是根据排序值来获取的，而且必须保证排序值是唯一的，es在返回结果的时候会把排序值一起返回，只有保证排序值是唯一的才能保证 search after 是准确的

step 2

使用上一步最后一个文档的 sort 值进行查询

比如第一步正常获取

```
# search after
GET test_search_index/_search
{
  "size": 1,
  "sort": {
    "age": "desc",
    "_id": "desc"
  }
}
```

![](https://images.notes.xuepincat.com/elastic/elasticsearch/97.png)

第二步将返回的排序值放到第二个查询中

```
GET test_search_index/_search
{
  "size": 1,
  "search_after": [
    28,
    "2"
  ],
  "sort": {
    "age": "desc",
    "_id": "desc"
  }
}
```

![](https://images.notes.xuepincat.com/elastic/elasticsearch/98.png)

然后再把发挥的 sort 放到第三次查询中 

```
GET test_search_index/_search
{
  "size": 1,
  "search_after": [
    23,
    "4"
  ],
  "sort": {
    "age": "desc",
    "_id": "desc"
  }
}
```

![](https://images.notes.xuepincat.com/elastic/elasticsearch/99.png)

* search_after 执行原理

如何避免from+size深度问题？

通过唯一排序值定位文档，将每次要处理的文档数控制在size内

比如有5个分片 p0,p1,p2,p3,p4,比如现在要获取下一页数据的时候，因为有一个排序值，每个分片只需要返回再这个排序值后面的10个文档，这里的10就是每次要获取的文档数，比如size:10,每个分片返回10个文档，Coordinating Node 只需要处理50个文档就可以了，然后将50个文档排序，最后返回10个文档就可以了

search_after 每次都将要处理的文档数控制在了一个 size 内，这样它的深度分页问题就没有了，而且性能也比较高，但是缺点就是只能获取下一页，不能指定访问页数

### 应用场景

|   类型  |   场景    |
|   :----:  |   :----:   |
|   from+size   |   需要实时获取顶部的部分文档，且需要自由翻页    |
|   scroll   |   需要全部文档，如导出所有数据的功能 |
|   search_after   |   需要全部文档，不需要自由翻页    |   
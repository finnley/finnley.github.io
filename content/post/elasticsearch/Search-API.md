+++
title = 'Search API'
date = 2020-05-28T00:37:46+08:00
draft = true
categories = [ "ElasticSearch" ]
tags = [ "es" ]
+++

## Search API

实现对 es 存储的数据进行查询分析，endpoint 为 _search 

可以指定索引查询，也可以一次查询多个

```
GET /_search
GET /my_index/_search
GET /my_index1, my_index2/_search
GET /my_*/_search
```

<!-- more -->

查询主要有两种形式

1. URI Search

操作简单，方便通过命令行测试

仅包含部分查询语法

```
GET /my_index/_search?q=user:alfred
```

2. Request Body Search

es 提供的完备查询语法 Query DSL (Doman Specific Language)

```
GET /my_index/_search
{
    "query": {
        "term": {
            "user": "alfred"
        }
    }
}
````
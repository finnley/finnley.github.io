+++
title = 'URI Search'
date = 2020-05-28T23:20:50+08:00
draft = true
categories = [ "ElasticSearch" ]
tags = [ "es" ]
+++

通过 url query 参数来实现搜索，常用参数如下：

- q 指定查询的语句，语法为 Query String Syntax

- df (default field)q中不指定字段时默认查询的字段，如果不指定，es会查询所有字段

- sort 排序

- timeout 指定超时时间，默认不超时，如果在指定时间内没有返回结果就会以超时报错

- from,size 用于分页

<!-- more -->

* URI Search

```
GET /my_index/_search?q=alfred&df=user&sort=age:asc&from=4&size=10&timeout=1s
```

查询 user 字段包含 alfred 的文档，结果按照 age 升序排列，返回第5~14个文档，如果超过1s没有结束，则以超时结束

## Query String Syntax

#### term 与 phrase

- alfred way 等效于 alfred OR way

term 是一个一个单词，如果查询的是 `alfred way` 中间是一个空格，其实等等效于查询 `alfred` OR `way` 这两个 term,只要包含某一个就表明是符合查询需求的

- "alfred way" 词语查询，要求先后顺序

phrase 是一个词语，如果词语查询，要用双引号包含起来，比如 `"alfred way"` 就以为这是词语查询，对于里面的 term 是有顺序要求的

#### 范查询

- alred 等效于所有字段去匹配该 term

不指定字段查询，前提没有指定 df 参数

#### 指定字段

- name:alfred

#### Group 分组设定，使用括号指定匹配的规则

- (quick OR brown) AND fox

括号的作用就是把词进行分组，通过括号指定了匹配的优先级，比如上面就要先判断前面的条件，然后再去判断后面的条件

- status:(active OR pending) title:(full text search)

括号的另一个作用是将关键词作为一个整体，上面就是返回 staus 是 active 或者 pending 的所有文档，如果不加括号的就是返回 status 是 active,或者所有字段中包含 pending 的文档;后面的如果不加括号就是返回 title 是 full,其他所有字段包含 text 或者 search 的文档

* 测试数据

```
# data
DELETE test_search_index

PUT test_search_index
{
  "settings": {
    "index":{
        "number_of_shards": "1"
    }
  }
}

# create test doc
POST test_search_index/doc/_bulk
{"index":{"_id":"1"}}
{"username":"alfred way","job":"java engineer","age":18,"birth":"1990-01-02","isMarried":false}
{"index":{"_id":"2"}}
{"username":"alfred","job":"java senior engineer and java specialist","age":28,"birth":"1980-05-07","isMarried":true}
{"index":{"_id":"3"}}
{"username":"lee","job":"java and ruby engineer","age":22,"birth":"1985-08-07","isMarried":false}
{"index":{"_id":"4"}}
{"username":"alfred junior way","job":"ruby engineer","age":23,"birth":"1989-08-07","isMarried":false}
```

* search api

1. 范查询

返回所有字段都包含 alfred 这个term的文档

```
GET test_search_index/_search?q=alfred
```

- response 

```
{
  "took": 22,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 3,
    "max_score": 1.2039728,
    "hits": [
      {
        "_index": "test_search_index",
        "_type": "doc",
        "_id": "2",
        "_score": 1.2039728,
        "_source": {
          "username": "alfred",
          "job": "java senior engineer and java specialist",
          "age": 28,
          "birth": "1980-05-07",
          "isMarried": true
        }
      },
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
        }
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
        }
      }
    ]
  }
}
```

范查询是查询了所有字段，es提供了profile的参数，可以做查询语句调优的一种手段，通过这个也可以去看es是怎么执行查询的条件的

```
GET test_search_index/_search?q=alfred
{
  "profile": true
}
```

2. 按照字段查询

查询 username 包含 alfred 的文档

```
GET test_search_index/_search?q=username:alfred
```

返回结果和上面一样

```
GET test_search_index/_search?q=username:alfred way
```

返回结果和上面一样,原因可以通过下面的profile查看，username:alfred 和 way是一个 `OR` 的关系，只要满足任何条件都可以

```
GET test_search_index/_search?q=username:alfred way
{
  "profile": true
}
```

3. 词语查询

可以给词语加上双引号

只返回了 username 是 `alfred way` 的一个文档

```
GET test_search_index/_search?q=username:"alfred way"
{
  "profile": true
}
```

还有一种方式加括号

```
GET test_search_index/_search?q=username:(alfred way)
{
  "profile": true
}
```

#### 布尔查询

布尔操作符

- AND(&&), OR(||), NOT(!)

name:(tom NOT lee) 表示在name里面不要有 lee, 但是可以包含 tom 的所有文档

主要大写，不能小写

```
GET test_search_index/_search?q=username:alfred AND way
{
  "profile": true
}
```

username 包含 alfred 和 way 的文档

```
GET test_search_index/_search?q=username:(alfred AND way)
{
  "profile": true
}
```

加上括号结果和上面结果是一样的

```
GET test_search_index/_search?q=username:(alfred NOT way)
{
  "profile": true
}
```

返回 username 中没有way,可以有 alfred 的所有文档

- `+` `-`分别对应 must 和 must_not

name:(tom +lee -alfred) 表示name一定包含lee,一定不包含alfred的，可以包含tom的所有文档

name:((lee && !alfred) || (tom && lee && !alfred))

`+` 在 url 中会被解析为空格，要使用 encode 后的结果才可以，为 `%2B`

```
GET test_search_index/_search?q=username:(alfred +way)
{
  "profile": true
}
```

- response

```
{
  "took": 1,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 3,
    "max_score": 0.99185646,
    "hits": [
      {
        "_index": "test_search_index",
        "_type": "doc",
        "_id": "1",
        "_score": 0.99185646,
        "_source": {
          "username": "alfred way",
          "job": "java engineer",
          "age": 18,
          "birth": "1990-01-02",
          "isMarried": false
        }
      },
      {
        "_index": "test_search_index",
        "_type": "doc",
        "_id": "4",
        "_score": 0.81242514,
        "_source": {
          "username": "alfred junior way",
          "job": "ruby engineer",
          "age": 23,
          "birth": "1989-08-07",
          "isMarried": false
        }
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
        }
      }
    ]
  }
}
```

会发现有一个条记录只有 alfred ,但是没有 way,这个因为 `+` 在起作用，`+` 被解析成了空格,需要使用 `%2B`，修改如下：

```
GET test_search_index/_search?q=username:(alfred +way)
{
  "profile": true
}
```

- response 

```
{
  "took": 1,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 2,
    "max_score": 0.99185646,
    "hits": [
      {
        "_index": "test_search_index",
        "_type": "doc",
        "_id": "1",
        "_score": 0.99185646,
        "_source": {
          "username": "alfred way",
          "job": "java engineer",
          "age": 18,
          "birth": "1990-01-02",
          "isMarried": false
        }
      },
      {
        "_index": "test_search_index",
        "_type": "doc",
        "_id": "4",
        "_score": 0.81242514,
        "_source": {
          "username": "alfred junior way",
          "job": "ruby engineer",
          "age": 23,
          "birth": "1989-08-07",
          "isMarried": false
        }
      }
    ]
  }
}
```

#### 范围查询，支持数值和日期

- 区间写法，闭区间用 `[]`, 开区间用 `{}`

age:[1 TO 10] 表示 `1 <= age <= 10`

age:[1 TO 10} 表示 `1 <= age < 10`

age:[1 TO ] 表示 `age >= 1`

age:[* TO 10] 表示 `age <= 10`

- 算数符号写法

age:>=1

age:(>=1 && <= 10) 或者 age:(+>=1 +<=10)

```
GET test_search_index/_search?q=username:alfred age:>20
{
  "profile": true
}
```

username 里面包含 alfred 的，或者 age 大于 20 的文档

```
GET test_search_index/_search?q=username:alfred AND age:>20
{
  "profile": true
}
```

username 里面包含 alfred 并且 age 大于 20 的文档

- request 出生日期在 1990 之前在 1980之后的文档

```
GET test_search_index/_search?q=birth:(>1980 AND <1990)
{
  "profile": true
}
```

#### 通配符查询

- `?` 代表1个字符， `*` 代表0或多个字符

name:t?m

name:tom*

name:t*m

- 通配符匹配执行效率低，且占用较多内存，不建议使用

- 如无特殊需求，不要将 `?/*` 放在最前面，表示需要匹配所有的文档，这个效率是最低的，很有可能将内存吃光

```
GET test_search_index/_search?q=username:alf*
```

查询以 `alf` 开头的所有文档

#### 正则表达式

- name:/[mb]oat/ 表示返回name里面包含 `moat` 或者 `boat` 的文档

正则表达式匹配也比较吃内存，不建议使用，尤其当文档数非常大的时候

```
GET test_search_index/_search?q=username:/[a]?l.*/
```

表示 username 里面可以是 `a` 开头，然后接着是 `l`, `l` 后面是任意字符，结果返回了 `lee`, 因为 `l` 前面是 `?`,表示 `a` 开头的可有可无

#### 模糊匹配 fuzzy query

- name:roam~1

匹配与 road 差1个 character 的词，比如 foam roams 等

```
GET test_search_index/_search?q=username:alfed~1
GET test_search_index/_search?q=username:alfd~2
```

##### 近似度查询 proximity search

- "fox quick" ~5

- 以 term 为单位进行差异比较，比如 "quick fox" "quick brown fox" 都会被匹配

```
GET test_search_index/_search?q=job:"java engineer"

GET test_search_index/_search?q=job:"java engineer" ~1

GET test_search_index/_search?q=job:"java engineer" ~2
```

## Request Body Search

将查询语句通过 http reqeust body 发送到 es，主要包含如下参数

- query 符合 Query DSL 语法的查询语句

- from, size

- timeout

- sort

- ...

比如：

- Request Body Search

```
GET /my_index/_search
{
    "query": {
        "term": {
            "user": "alfred"
        }
    }
}
```

#### Query DSL

基于 JSON 定义的查询语言，主要包含如下两种类型：

- 字段类查询

如 term, match, range 等，只针对某一个字段进行查询

term: 针对词的查询
match: 针对全文检索的查询
range: 针对范围查询

- 复合查询

如 bool 查询等，包含一个或多个字段类查询或者复合查询语句

## 字段查询

字段查询主要包括两类

1. 全文匹配

针对 text 类型的字段进行全文检索，会对查询的语句先进行分词处理，然后再拿分词的结果去es中存的倒排索引的term做匹配，如 match, match_parase 等query 类型

2. 单词匹配

不会对查询语句做分词处理，直接去匹配字段的倒排索引，如 term, terms, range 等 query 类型

#### match query

对字段做全文检索，最基本和常用的查询类型

- request: 返回 username 里面有 alfred 或者 way 的文档

```
GET test_search_index/_search
{
  "profile": true,
  "query": {
    "match": {
      "username": "alfred way"
    }
  }
}
```

#### match query 执行流程

首先会对查询语句进行分词，比如有一个查询语句 `alfred way` ，分词之后会被分为 `alfred` 和 `way`,es 会拿到username 的倒排索引对 alfred 和 way 进行匹配算分，常见的算分算法有 `TF/IDF` 和 `BM25`,最终es会列出文档和查询语句匹配的得分，然后es会对文档的得分结果进行汇总，根据得分排序，返回匹配文档

alfred 和 way 是两个 term, es对这两个term处理的时候是一个什么关系呢？是只有一个即可还是必须同时存在？其实是只要有一个存在即可，如果必须两个词都存在应该怎么做呢？es提供了一个 `operator` 参数

通过 operator 参数可以控制单词间的匹配关系，可选为 `or` 和 `and`

username 里面必须包含 alfred 和 way 这两个 term 的结果
```
GET test_search_idnex/_search
{
    "query": {
        "match": {
            "username": {
                "query": "alfred way",
                "operator": "and"
            }
        }
    }
}
```

通过 `minimum_should_match` 参数可以控制需要匹配的单词数

```
GET test_search_index/_search
{
  "query": {
    "match": {
      "username": {
        "query": "alfred way",
        "minimum_should_match": "2"
      }
    }
  }
}
```

```
GET test_search_index/_search
{
  "query": {
    "match": {
      "job": {
        "query": "java ruby engineer",
        "minimum_should_match": "2"
      }
    }
  }
}
```

## 相关性算分

相关性算分是指文档与查询语句间的相关度，英文为 relevance

- 通过倒排索引可以获取与查询语句相关匹配的文档列表，那么如何将最符合用户查询需求的文档排到前面呢？

- 本质是一个排序问题，排序的一句是相关性算分

现有一个倒排索引

|   单词  |   文档ID列表    |
|   :----:  |   :----:   |
|   alfred   |   1,2   |
|   way    |   1   |

通过倒排索引拿到一个term就能知道term对应在哪些文档中出现过，比如 `alfred` 有两个文档1和2，对于1和2两个文档在返回给用户的时候应该把1排在前面还是把2排在前面，到底是文档1还是文档2更符合用户期望的结果呢？这里就涉及到相关性算分，相关性算分本质解决的就是文档返回结果的排序问题

相关性算分的几个重要概念

* Term Frequency(TF) 词频，即单词在该文档中出现的次数。词频越高，相关度越高

* Document Frequency(DF) 文档频率，即单词出现的文档数

* Inverse Document Frequency(IDF) 逆向文档频率，与文档频率相反，简单理解为 `1/DF`。即单词出现的文档数越少，相关度越高

* Field-length Norm 文档越短，相关性越高

ES 有两个主要的相关性算分模型

* TF/IDF 模型

* BM25 模型，5.x之后的默认模型

可以通过 `explain` 参数来查看具体的计算方法

es 的算法按照 shard 进行的，即 shard 的分数计算是相互独立的，所以在使用 explain 的时候需要注意分片数

可以通过设置的索引的分片数为 1 来避免这个问题

```
GET test_search_index/_search
{
  "explain": true,
  "query": {
    "match": {
      "username": "alfred way"
    }
  }
}
```

```
PUT test_search_index
{
  "settings": {
    "index": {
      "number_of_shards": 1
    }
  }
}
```

## match phrase query

对字段作检索，有顺序要求

- request

```
GET test_search_index/_search
{
  "query": {
    "match_phrase": {
      "job": "java engineer"
    }
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
    "total": 1,
    "max_score": 0.5602635,
    "hits": [
      {
        "_index": "test_search_index",
        "_type": "doc",
        "_id": "1",
        "_score": 0.5602635,
        "_source": {
          "username": "alfred way",
          "job": "java engineer",
          "age": 18,
          "birth": "1990-01-02",
          "isMarried": false
        }
      }
    ]
  }
}
```

可以通过 `slop` 参数控制单词间的间隔

- request

```
GET test_search_index/_search
{
  "query": {
    "match_phrase": {
      "job": {
        "query": "java engineer",
        "slop": "1"
      }
    }
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
    "total": 2,
    "max_score": 0.5602635,
    "hits": [
      {
        "_index": "test_search_index",
        "_type": "doc",
        "_id": "1",
        "_score": 0.5602635,
        "_source": {
          "username": "alfred way",
          "job": "java engineer",
          "age": 18,
          "birth": "1990-01-02",
          "isMarried": false
        }
      },
      {
        "_index": "test_search_index",
        "_type": "doc",
        "_id": "2",
        "_score": 0.21693127,
        "_source": {
          "username": "alfred",
          "job": "java senior engineer and java specialist",
          "age": 28,
          "birth": "1980-05-07",
          "isMarried": true
        }
      }
    ]
  }
}
```

## query-string-query

类似于 URI Search 中的 q 参数查询

- request

```
GET test_search_index/_search
{
  "query": {
    "query_string": {
      "default_field": "username",
      "query": "alfred AND way"
    }
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
    "total": 2,
    "max_score": 0.99185646,
    "hits": [
      {
        "_index": "test_search_index",
        "_type": "doc",
        "_id": "1",
        "_score": 0.99185646,
        "_source": {
          "username": "alfred way",
          "job": "java engineer",
          "age": 18,
          "birth": "1990-01-02",
          "isMarried": false
        }
      },
      {
        "_index": "test_search_index",
        "_type": "doc",
        "_id": "4",
        "_score": 0.81242514,
        "_source": {
          "username": "alfred junior way",
          "job": "ruby engineer",
          "age": 23,
          "birth": "1989-08-07",
          "isMarried": false
        }
      }
    ]
  }
}
```

- request 使用 fields 执行查询字段,

```
GET test_search_index/_search
{
  "query": {
    "query_string": {
      "fields": [
        "username",
        "job"
      ],
      "query": "alfred OR (java AND ruby)"
    }
  }
}
```

查询 username 和 job, 指定字段可以包含 alfred 或者同时包含 java 和 ruby 的文档

- response

```
{
  "took": 1,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 4,
    "max_score": 0.99185646,
    "hits": [
      {
        "_index": "test_search_index",
        "_type": "doc",
        "_id": "3",
        "_score": 0.99185646,
        "_source": {
          "username": "lee",
          "job": "java and ruby engineer",
          "age": 22,
          "birth": "1985-08-07",
          "isMarried": false
        }
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
        }
      },
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
        }
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
        }
      }
    ]
  }
}
```

## simple-query-string-query

类似 Query String ,但是会忽略错误的查询语法，并且仅支持部分查询语法

其常用的逻辑符号如下，不能使用 AND, OR, NOT 等关键词：

* `+` 代指 AND

* `|` 代指 OR

* `-` 代指 NOT

- request

```
GET test_search_index/_search
{
  "query": {
    "simple_query_string": {
      "query": "alfred +way",
      "fields": [
        "username"
      ]
    }
  }
}
```

## term-terms-query

* term

将查询语句作为整个单词进行查询，即不对语句做分词梳理

- request

```
GET test_search_index/_search
{
  "query": {
    "term": {
      "username": "alfred"
    }
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
    "total": 3,
    "max_score": 0.43250346,
    "hits": [
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
        }
      },
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
        }
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
        }
      }
    ]
  }
}
```

* terms

一次传入多个单词进行查询

```
GET test_search_index/_search
{
  "query": {
    "terms": {
      "username": [
        "alfred",
        "way"
      ]
    }
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
    "total": 3,
    "max_score": 1,
    "hits": [
      {
        "_index": "test_search_index",
        "_type": "doc",
        "_id": "1",
        "_score": 1,
        "_source": {
          "username": "alfred way",
          "job": "java engineer",
          "age": 18,
          "birth": "1990-01-02",
          "isMarried": false
        }
      },
      {
        "_index": "test_search_index",
        "_type": "doc",
        "_id": "2",
        "_score": 1,
        "_source": {
          "username": "alfred",
          "job": "java senior engineer and java specialist",
          "age": 28,
          "birth": "1980-05-07",
          "isMarried": true
        }
      },
      {
        "_index": "test_search_index",
        "_type": "doc",
        "_id": "4",
        "_score": 1,
        "_source": {
          "username": "alfred junior way",
          "job": "ruby engineer",
          "age": 23,
          "birth": "1989-08-07",
          "isMarried": false
        }
      }
    ]
  }
}
```

## range-query

范围查询，主要针对数值和日期类型

- request

```
GET test_search_index/_search
{
  "query": {
    "range": {
      "age": {
        "gte": 10,
        "lte": 20
      }
    }
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
    "total": 1,
    "max_score": 1,
    "hits": [
      {
        "_index": "test_search_index",
        "_type": "doc",
        "_id": "1",
        "_score": 1,
        "_source": {
          "username": "alfred way",
          "job": "java engineer",
          "age": 18,
          "birth": "1990-01-02",
          "isMarried": false
        }
      }
    ]
  }
}
```

* Date Math

针对日期做查询

now - 1d

`now`: 基准日期，也可以是具体的日期，比如 2020-01-01，使用具体的日期的时候要用 `||` 做隔离

`- 1d`: 计算公式，主要有如下3中：
+1h 加1个小时
-1d 减1天
/d 将时间舍入到天

单位主要有如下几种:

- y - year
- M - months
- w - weeks
- d - days
- h - hours
- m - minutes
- s - seconds

假设 now 为 2020-01-02 12:00:00, 那么如下的计算结果实际为：

|   计算公式  |   实际结果    |
|   :----:  |   :----:   |
|   now+1h   |   2020-01-02 13:00:00   |
|   now-1h    |   2020-01-02 11:00:00   |
|   now-1h/d    |   2018-01-02 00:00:00   |
|   2018-01-01||+1M/d    |   2018-02-01 00:00:00  |

- request

```
GET test_search_index/_search
{
  "query": {
    "range": {
      "birth": {
        "gte": "1990-01-01"
      }
    }
  }
}
```

- response

```
{
  "took": 1,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 1,
    "hits": [
      {
        "_index": "test_search_index",
        "_type": "doc",
        "_id": "1",
        "_score": 1,
        "_source": {
          "username": "alfred way",
          "job": "java engineer",
          "age": 18,
          "birth": "1990-01-02",
          "isMarried": false
        }
      }
    ]
  }
}
```

- request

```
GET test_search_index/_search
{
  "query": {
    "range": {
      "birth": {
        "gte": "now-30y"
      }
    }
  }
}
```

- request

```
GET test_search_index/_search
{
  "query": {
    "range": {
      "birth": {
        "gte": "2010||-20y"
      }
    }
  }
}
```

## 复合查询

复合查询是指包含字段类查询或复合查询的类型，主要包括下面几类：

- constant_score query

- bool query

- dis_max query

- function_score query

- boosting query

#### Constant Score Query

该查询将其内部的查询结果文档得分都设定为1或者boost的值，多用于结合 bool 查询实现自定义得分

- request

```
GET test_search_index/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "match": {
          "username": "alfred"
        }
      },
      "boost": 1.2
    }
  }
}
```

constant_score 为关键词
filter 只能有一个

## Bool Query

布尔查询由一个或多个子句组成，主要包含如下4个

|   :----:  |   :----:   |
|   filter   |   只过滤复合条件的文档，不计算相关性得分   |
|   must    |   文档必须复合 must 中的所有条件，会影响相关性得分   |
|   must_not    |   文档必须不复合 must_not 中的所有条件   |
|   should    |   文档可以符合 should 中的条件，会影响相关性得分  |

* Filter

Filter 查询值过滤复合条件的文档，不会进行相关性算分

es 针对 filter 会有智能缓存，不会进行相关性算分

做简单匹配查询且不考虑算分时，推荐使用 filter 替代 query 等，索引在计算相关性得分的时候回非常耗时，如果将这个省去，在性能和查询速度会得到很大提升

- request

```
# filter query
GET test_search_index/_search
{
  "query": {
    "bool": {
      "filter": {
        "term": {
          "username": "alfred"
        }
      }
    }
  }
}
```
* Must

must 会影响文档的相关性算分，比如must中有两个match,2个match query 文档最终得分为这两个查询的得分加和，在根据加和的结果做最终排序

- reqeust

```
GET test_search_index/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "username": "alfred"
          }
        },
        {
          "match": {
            "job": "specialist"
          }
        }
      ]
    }
  }
}
```

* Must Not

- request

```
GET test_search_index/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "job": "java"
          }
        }
      ],
      "must_not": {
        "match": {
          "job": "ruby"
        }
      }
    }
  }
}
```

* Should

shoule 使用分为两种情况

1. bool 查询中只包含 should, 不包含 must 查询

必须满足至少一个条件，当然可以设置 `minimum_should_match 一个条件` 来控制满足条件的个数

- request

```
GET test_search_index/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "match": {
            "username": "junior"
          }
        },
        {
          "match": {
            "job": "ruby"
          }
        }
      ]
    }
  }
}
```

- request

```
GET test_search_index/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "term": {
            "job": "java"
          }
        },
        {
          "term": {
            "job": "ruby"
          }
        },
        {
          "term": {
            "job": "specialist"
          }
        }
      ],
      "minimum_should_match": 2
    }
  }
}
```

minimum_should_match 表示至少满足2个条件


2. bool 查询中同时包含 should 和 must 查询

同时包含 should 和 must 时，文档不必满足should中的条件，但是如果满足条件，会增加相关性得分

查询 username 包含 alfred 的文档，同时将 job 包含 ruby 的文档排在前面

- request

先查询没有should

```
GET test_search_index/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "username": {
              "value": "alfred"
            }
          }
        }
      ]
    }
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
    "total": 3,
    "max_score": 0.43250346,
    "hits": [
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
        }
      },
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
        }
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
        }
      }
    ]
  }
}
```

此时的文档顺序按 _id: 2, 1, 4

- request

```
GET test_search_index/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "username": {
              "value": "alfred"
            }
          }
        }
      ],
      "should": [
        {
          "term": {
            "job": {
              "value": "ruby"
            }
          }
        }
      ]
    }
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
    "total": 3,
    "max_score": 1.116529,
    "hits": [
      {
        "_index": "test_search_index",
        "_type": "doc",
        "_id": "4",
        "_score": 1.116529,
        "_source": {
          "username": "alfred junior way",
          "job": "ruby engineer",
          "age": 23,
          "birth": "1989-08-07",
          "isMarried": false
        }
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
        }
      },
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
        }
      }
    ]
  }
}
```

文档的顺序变了，按照 _id: 4, 2, 1

#### Query Context VS Filter Context

当一个查询语句位于 Query 或者 Filter 上下文时，es执行的结果会不同，对比如下：

|   上下文类型  |   执行类型    |   使用方式    |
|   :----:  |   :----:   |   :----:   |
|   Query   |   查找与查询语句最匹配的文档，对所有文档进行相关性算分并排序   |   query;
bool 中的 must 和 should   |
|   Filter    |   查找与查询语句相匹配的文档   |   bool 中的 filter 与 must_not; constant_score 中的 filter   |

```
GET test_search_index/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "title": "Search"
          }
        },
        {
          "match": {
            "content": "Elasticsearch"
          }
        }
      ],
      "filter": [
        {
          "term": {
            "status": "published"
          }
        },
        {
          "range": {
            "publish_date": {
              "gte": "2020-01-01"
            }
          }
        }
      ]
    }
  }
}
```

must 里面是 query 上下文，会进行先关性算分

filter 里面是 filter 上下文，不会影响算分，智慧过滤复合条件的文档，实际使用中可以把不需要做相关性算分的字段都放在 filter 里面

## Count

获取复合条件的文档数，endpoint 为 _count

- request

```
GET test_search_index/_count
{
  "query": {
    "match": {
      "username": "alfred"
    }
  }
}
```

- response

```
{
  "count": 3,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  }
}
```

## Source Filtering

过滤返回结果中的 `_source` 中的字段，主要有如下几种方式：

* url 参数

```
GET test_search_index/_search?_source=username
```

* 不返回 _source

```
GET test_search_index/_search
{
  "_source": false
}
```

* 返回部分字段

```
GET test_search_index/_search
{
  "_source": ["username", "age"]
}
```


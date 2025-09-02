+++
title = 'Mapping'
date = 2020-05-24T00:01:17+08:00
draft = true
categories = [ "ElasticSearch" ]
tags = [ "es" ]
+++

## 什么是 Mapping

类似数据库中表结构定义，主要作用如下：
    
    - 定义 Index 下的字段名（Field Name）
    - 定义字段的类型，比如数值型，字符串型，布尔型等
    - 定义倒排索引相关配置，比如是否索引，记录 postion 等

- request

```
GET test_index/_mapping
```

- response

```
{
  "test_index": {
    "mappings": {
      "doc": {
        "properties": {
          "age": {
            "type": "long"
          },
          "username": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          }
        }
      }
    }
  }
}
```

## 自定义 mapping

自定义 Mapping API 

mappings: mappings 关键词
doc: type名称
properties: 字段名称和类型定义
type: 字段类型 

- request

```
PUT my_index
{
  "mappings": {
    "doc": {
      "properties": {
        "title": {
          "type": "text"
        },
        "name": {
          "type": "keyword"
        },
        "age": {
          "type": "integer"
        }
      }
    }
  }
}
```

- response

```
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "index": "my_index"
}
```

![](https://images.notes.xuepincat.com/elastic/elasticsearch/35.png)

* Mapping 中的字段类型一旦设定后，禁止直接修改

* 如果要改的话就必须重新建立新的索引，然后做 reindex 操作，reindex就是将所有的文档重新导入到新的索引里面

原因：Lucence 实现的倒排索引生成后不允许修改，Lucence 为了提高效率，限定了生成的索引一旦生成不允许修改

* 允许新增字段

    新增的时候因为类型不定，新增的设置好字段类型，就可以用了，对于es来说就是增加了倒排索引

    可以通过 dynamic 参数来控制字段的新增

    - true (默认) 允许自动新增字段
    - false 不允许自动新增字段，如果文档里面字段是mapping中没有定义的，文档可以正常写入，但是字段不会在mapping中新增出来，意味着无法对这个字段进行查询等操作
    - strict 严格模式，如果文档中出现了mapping中没有定义过的字段，写入就会报错

- request

```
# mapping
PUT my_index_test
{
  "mappings": {
    "doc": {
      "dynamic": false,
      "properties": {
        "title": {
          "type": "text"
        },
        "name": {
          "type": "keyword"
        },
        "age": {
          "type": "integer"
        }
      }
    }
  }
}
```

- response

```
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "index": "my_index_test"
}
```

![](https://images.notes.xuepincat.com/elastic/elasticsearch/36.png)

查看 mapping

- request

```
GET my_index_test/_mapping
```

- response

```
{
  "my_index_test": {
    "mappings": {
      "doc": {
        "dynamic": "false",
        "properties": {
          "age": {
            "type": "integer"
          },
          "name": {
            "type": "keyword"
          },
          "title": {
            "type": "text"
          }
        }
      }
    }
  }
}
```

![](https://images.notes.xuepincat.com/elastic/elasticsearch/37.png)

现在写入一个文档

- request

```
PUT my_index_test/doc/1
{
  "title": "hello,world",
  "desc": "nothing here"
}
```

- response

```
{
  "_index": "my_index_test",
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

![](https://images.notes.xuepincat.com/elastic/elasticsearch/38.png)

查询文档

- request

```
GET my_index_test/doc/_search
{
  "query": {
    "match": {
      "title": "hello"
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
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 0.2876821,
    "hits": [
      {
        "_index": "my_index_test",
        "_type": "doc",
        "_id": "1",
        "_score": 0.2876821,
        "_source": {
          "title": "hello,world",
          "desc": "nothing here"
        }
      }
    ]
  }
}
```

![](https://images.notes.xuepincat.com/elastic/elasticsearch/39.png)

但是此时去查询desc

- request

```
GET my_index_test/doc/_search
{
  "query": {
    "match": {
      "desc": "here"
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
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 0,
    "max_score": null,
    "hits": []
  }
}
```

![](https://images.notes.xuepincat.com/elastic/elasticsearch/40.png)

也就是说 desc 是没法去查询的，现在的mapping中字段还是和之前一样，并没有新增 desc

* strict 模式

- request

```
DELETE my_index

# mapping
PUT my_index
{
  "mappings": {
    "doc": {
      "dynamic": "strict",
      "properties": {
        "title": {
          "type": "text"
        },
        "name": {
          "type": "keyword"
        },
        "age": {
          "type": "integer"
        }
      }
    }
  }
}
```

- response

```
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "index": "my_index"
}
```

![](https://images.notes.xuepincat.com/elastic/elasticsearch/41.png)

创建文档

- request

```
PUT my_index_test/doc/1
{
  "title": "hello,world",
  "desc": "nothing here"
}
```

- response

```
{
  "error": {
    "root_cause": [
      {
        "type": "strict_dynamic_mapping_exception",
        "reason": "mapping set to strict, dynamic introduction of [desc] within [doc] is not allowed"
      }
    ],
    "type": "strict_dynamic_mapping_exception",
    "reason": "mapping set to strict, dynamic introduction of [desc] within [doc] is not allowed"
  },
  "status": 400
}
```
![](https://images.notes.xuepincat.com/elastic/elasticsearch/42.png)

发现报错了

## copy_to

将该字段的值复制到目标字段，实现类似 6.0 之前 `_all` 的作用

不会出现在 `_source` 中，只用来搜索

- request

```
# copy_to
DELETE my_index

PUT my_index
{
  "mappings": {
    "doc": {
      "properties": {
        "first_name": {
          "type": "text",
          "copy_to": "full_name"
        },
        "last_name": {
          "type": "text",
          "copy_to": "full_name"
        },
        "full_name": {
          "type": "text"
        }
      }
    }
  }
}
```

- response

```
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "index": "my_index"
}
```

![](https://images.notes.xuepincat.com/elastic/elasticsearch/43.png)

新建文档

- request

```
PUT my_index/doc/1
{
  "first_name": "John",
  "last_name": "Smith"
}
```

- response

```
{
  "_index": "my_index",
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

![](https://images.notes.xuepincat.com/elastic/elasticsearch/44.png)

查询文档

- request

```
GET my_index/doc/_search
{
  "query": {
    "match": {
      "full_name": {
        "query": "John Smith",
        "operator": "and"
      }
    }
  }
}
```

- response

```
{
  "took": 7,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 0.5753642,
    "hits": [
      {
        "_index": "my_index",
        "_type": "doc",
        "_id": "1",
        "_score": 0.5753642,
        "_source": {
          "first_name": "John",
          "last_name": "Smith"
        }
      }
    ]
  }
}
```

![](https://images.notes.xuepincat.com/elastic/elasticsearch/45.png)

## index

控制当前字段是否索引，默认为true,即记录索引，false不记录，即不可搜索

当在 es 中存了一些字段，这些字段是不需要被查询的，或者不希望这些字段被用来查询，比如一些敏感信息，就可以将 index 设置 false,既可以满足不被检索的需求，也可以节省空间，因为不需要存倒排索引了，就会节省磁盘和内存空间

- request

```
# index
DELETE my_index

PUT my_index
{
  "mappings": {
    "doc": {
      "properties": {
        "cookie": {
          "type": "text",
          "index": false
        }
      }
    }
  }
}
```

- response

```
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "index": "my_index"
}
```

![](https://images.notes.xuepincat.com/elastic/elasticsearch/46.png)

新建文档

- request

```
PUT my_index/doc/1
{
  "cookie": "name=alfred"
}
```

- response

```
{
  "_index": "my_index",
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

![](https://images.notes.xuepincat.com/elastic/elasticsearch/47.png)

查询文档

- request

```
GET my_index/_search
{
  "query": {
    "match": {
      "cookie": "name"
    }
  }
}
```

- response

```
{
  "error": {
    "root_cause": [
      {
        "type": "query_shard_exception",
        "reason": "failed to create query: {\n  \"match\" : {\n    \"cookie\" : {\n      \"query\" : \"name\",\n      \"operator\" : \"OR\",\n      \"prefix_length\" : 0,\n      \"max_expansions\" : 50,\n      \"fuzzy_transpositions\" : true,\n      \"lenient\" : false,\n      \"zero_terms_query\" : \"NONE\",\n      \"auto_generate_synonyms_phrase_query\" : true,\n      \"boost\" : 1.0\n    }\n  }\n}",
        "index_uuid": "0qNOdqXKSZW8E6Mg99iSvA",
        "index": "my_index"
      }
    ],
    "type": "search_phase_execution_exception",
    "reason": "all shards failed",
    "phase": "query",
    "grouped": true,
    "failed_shards": [
      {
        "shard": 0,
        "index": "my_index",
        "node": "wojtGw1HT3ihL0P88fwV7w",
        "reason": {
          "type": "query_shard_exception",
          "reason": "failed to create query: {\n  \"match\" : {\n    \"cookie\" : {\n      \"query\" : \"name\",\n      \"operator\" : \"OR\",\n      \"prefix_length\" : 0,\n      \"max_expansions\" : 50,\n      \"fuzzy_transpositions\" : true,\n      \"lenient\" : false,\n      \"zero_terms_query\" : \"NONE\",\n      \"auto_generate_synonyms_phrase_query\" : true,\n      \"boost\" : 1.0\n    }\n  }\n}",
          "index_uuid": "0qNOdqXKSZW8E6Mg99iSvA",
          "index": "my_index",
          "caused_by": {
            "type": "illegal_argument_exception",
            "reason": "Cannot search on field [cookie] since it is not indexed."
          }
        }
      }
    ]
  },
  "status": 400
}
```

![](https://images.notes.xuepincat.com/elastic/elasticsearch/48.png)

## index_options

用于控制如何记录倒排索引，以及倒排索引记录的内容，配置如下：

    - docs 只记录 doc id
    - freqs 记录 doc id 和 term frequencies (词频，单词在文档中出现的次数)
    - positions 记录 doc id,term frequencies, 和 term position （term在文档中是第几个位置出现，可以做 match 等词语的查询）
    - offsets 记录 doc id, term frequencies, term position 和 chatacter offsets，记录词在文档中开始和结束的位置，有了位置之后就可以做高亮

text 类型默认配置为 positions ,其他默认为 docs

记录内容越多，占用空间越大

* index_options

- request

```
# index_options
PUT my_index
{
  "mappings": {
    "doc": {
      "properties": {
        "cookie": {
          "type": "text",
          "index_options": "offsets"
        }
      }
    }
  }
}
```

* null_value

当字段遇到 null 值的时候处理策略，默认为 null, 即控制，此时es会忽略该值，可以通过设定该值设定字段的默认值

- request

```
# null_value
PUT my_index
{
  "mappings": {
    "my_type": {
      "properties": {
        "status_code": {
          "type": "keyword",
          "null_value": "NULL"
        }
      }
    }
  }
}
```

## 数据类型

#### 基本数据类型

  - 字符串型 text, keyword

  - 数值型 long, integer, short, byte, double, float, half_float, scaled_float

  - 日期类型 date

  - 布尔类型 boolean

  - 二进制类型 binary

  - 范围类型 integer_range, float_range, long_range, double_range, date_range

#### 复杂数据类型

  - 数组类型 array

  - 对象类型 object

  - 嵌套类型 nested object

#### 地理位置数据类型

  - geo_point

  - geo_shape

#### 专用类型

  - 记录IP地址 ip

  - 实现自动补全 completion

  - 记录分词数 token_count

  - 记录字符串 hash 值 murmur3

  - percolator

  - join (partne child 父子查询的)

#### 多字段特性 multi-fields

  - 允许对同一个字段采用不同的配置，比如分词，常见的如对人名实现拼音搜索，只需要在人名中新增一个子字段为 pinyin 即可，提前将 pinyin 取出来，索引的这个文档的时候将这个字段插入进去，查询的时候查询这个字段就可以了，这种实现并不优雅，因为已经影响了原来的数据结构，所以使用 multi-fields 就可以完美的解决这个问题，因为它不会修改原始的文档结构，只是在原始的文档结构下新增了一个子字段

```
{
  "text_index": {
    "mapping": {
      "doc": {
        "properties": {
          "username": {
            "type": "text",
            "fields": {
              "pinyin": {
                "type": "text",
                "analyzer": "pinyin"
            }
          }
        }
      }
    }
  }
}
```

查询

```
GET test_index/_search
{
  "query": {
    "match": {
      "username.pinyin": "hanhan"
    }
  }
}
```

## dynamic-mapping

es 可以自动识别文档字段类型，从而降低用户使用成本

- request

```
# dynamic mapping
DELETE test_index

PUT /test_index/doc/1
{
  "username": "alfred",
  "age": 1
}

GET /test_index/_mapping
```

- response

```
{
  "test_index": {
    "mappings": {
      "doc": {
        "properties": {
          "age": {
            "type": "long"
          },
          "username": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          }
        }
      }
    }
  }
}
```

![](https://images.notes.xuepincat.com/elastic/elasticsearch/49.png)

如果一开始索引不存在的话，es会自动帮我们创建索引，创建索引的时候一个字段的类型是需要指定的，es是可以自动识别文档字段的类型，不需要像MYSQL一样必须先将表结构设定好，es中直接写写数据就可以

#### 类型识别

es是如何自动识别字段类型的呢？其实是依靠JSON文档的字段类型，es通过JSON类型推导出对应的es类型

|   JSON 类型  |   es 类型    |
|   :----:  |   :----:   |
|   null   |   忽略   |
|   boolean    |   boolean   |
|   浮点类型    |   float |
|   整数    |   long |
|   object    |   object |
|   array    |   由第一个非 null 值的类型决定 |
|   string    |   匹配为日期则设为 date 类型 (默认开启);匹配为数字的话则设为float或long类型（默认关闭）；设为text类型，并附带keyword的子字段 |

- request

```
DELETE test_index

PUT /test_index/doc/1
{
  "username": "alfred",
  "age": 14,
  "birth": "1988-10-10",
  "married": false,
  "year": "18",
  "tags": ["boy", "fashion"],
  "money": 100.1
}

GET /test_index/_mapping
```

- response

```
{
  "test_index": {
    "mappings": {
      "doc": {
        "properties": {
          "age": {
            "type": "long"
          },
          "birth": {
            "type": "date"
          },
          "married": {
            "type": "boolean"
          },
          "money": {
            "type": "float"
          },
          "tags": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "username": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "year": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          }
        }
      }
    }
  }
}
```

#### 日期自动识别

日期的自动识别可以自定配置日期格式，以满足各种需求

- 默认是 `["strict_date_optional_time", "yyyy/MM/dd HH:mm:ss Z||yyyy/MM/dd Z"]`

strict_date_optional_time 是 ISO datetime 的是个，完整格式类似下面：

  YYYY-MM-DDYhh:mm:ssTZD (eg 1997-07-16T19:20:30+01:00)

- dynamic_date_formats 可以自定义日期类型

- date_detection es默认开了日期的自动识别，可以关闭日期自动识别的机制

* 自定义日期识别格式

- request

```
DELETE my_index

PUT my_index
{
  "mappings": {
    "my_type": {
      "dynamic_date_formats": ["MM/dd/yyyy"]
    }
  }
}

PUT my_index/my_type/1
{
  "create_date": "05/25/2020"
}

GET /test_index/_mapping
```

- response

```
{
  "my_index": {
    "mappings": {
      "my_type": {
        "dynamic_date_formats": [
          "MM/dd/yyyy"
        ],
        "properties": {
          "create_date": {
            "type": "date",
            "format": "MM/dd/yyyy"
          }
        }
      }
    }
  }
}
```

![](https://images.notes.xuepincat.com/elastic/elasticsearch/50.png)

* 关闭日期自动识别机制

```
PUT my_index
{
  "mappings": {
    "my_type": {
      "date_detection": false
    }
  }
}
```

#### 数字自动识别

字符串是数字时，默认不会自动识别为整数，因为字符串中出现数字时完全河里的

`numeric_detection` 可以开启字符串中数字的自动识别

- request

```
DELETE my_index

PUT my_index
{
  "mappings": {
    "my_type": {
      "numeric_detection": true
    }
  }
}

PUT my_index/my_type/1
{
  "my_float": "1.0",
  "my_integer": "1"
}

GET my_index/_mapping
```

- response

```
{
  "my_index": {
    "mappings": {
      "my_type": {
        "numeric_detection": true,
        "properties": {
          "my_float": {
            "type": "float"
          },
          "my_integer": {
            "type": "long"
          }
        }
      }
    }
  }
}
```

![](https://images.notes.xuepincat.com/elastic/elasticsearch/51.png)

## dynamic-template

允许根据es自动识别的数据类型，字段名等来动态设定字段类型，实现效果如下：

- 所有子豆腐串类型都设定为 keyword 类型，即默认部分词，

因为es识别字符串会设定为text,并且分词，而且都会加一个子字段，这个相对比较占空间，一般在实际使用的时候会默认将这个设定覆盖掉，也就是所有的字符串会设置成 keyword,不要分词，哪些字段需要分词单独去设置

- 所有以 message 开头的字段都设定为 text 类型，即分词

- 所有以 long_ 开头的字段都设定为 long 类型

- 所有自动匹配为 double 类型的都设定为 float 类型，以节省空间

```
PUT test_index
{
  "mappings": {
    "doc": {
      "dynamic_templates": [ # 数组，可以指定多个匹配规则
        {
          "string": { # template 的名称
            "match_mapping_type": "string", # 匹配规则
            "mapping": { # 设置mapping信息
              "type": "keyword"
            }
          }
        }
      ]
    }
  }
}
```

#### 匹配规则

匹配规则一般有如下几个参数：

- match_mapping_type 匹配es自动识别的字段类型，如boolean,long,string等

- match, unmatch 匹配字段名

- path_match, path_unmatch 匹配路径

#### 字符串默认使用 keyword 类型

es 默认会为字符串设置为text类型，并增加一个keyword的子字段

- request

```
# dynamic template
DELETE test_index

PUT test_index/doc/1
{
  "name": "alfred"
}

GET test_index/_mapping

PUT test_index
{
  "mappings": {
    "doc": {
      "dynamic_templates": [
        {
          "string_as_keywords": {
            "match_mapping_type": "string",
            "mapping": {
              "type": "keyword"
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
  "test_index": {
    "mappings": {
      "doc": {
        "dynamic_templates": [
          {
            "string_as_keywords": {
              "match_mapping_type": "string",
              "mapping": {
                "type": "keyword"
              }
            }
          }
        ]
      }
    }
  }
}
```

#### 以 message 开头的字段都设置为 text 类型

- request

```
# dynamic template
DELETE test_index

PUT test_index/doc/1
{
  "name": "alfred",
  "message": "handsome boy"
}

GET test_index/_mapping

PUT test_index
{
  "mappings": {
    "doc": {
      "dynamic_templates": [
        {
          "message_as_text": {
            "match_mapping_type": "string",
            "match": "message",
            "mapping": {
              "type": "text"
            }
          }
        },
        {
          "string_as_keywords": {
            "match_mapping_type": "string",
            "mapping": {
              "type": "keyword"
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
  "test_index": {
    "mappings": {
      "doc": {
        "dynamic_templates": [
          {
            "message_as_text": {
              "match": "message",
              "match_mapping_type": "string",
              "mapping": {
                "type": "text"
              }
            }
          },
          {
            "string_as_keywords": {
              "match_mapping_type": "string",
              "mapping": {
                "type": "keyword"
              }
            }
          }
        ],
        "properties": {
          "message": {
            "type": "text"
          },
          "name": {
            "type": "keyword"
          }
        }
      }
    }
  }
}
```

#### double 类型设置为 float ，可以节省空间

- request

```
PUT test_index
{
  "mappings": {
    "doc": {
      "dynamic_templates": [
        {
          "double_as_float": {
            "match_mapping_type": "double",
            "mapping": {
              "type": "float"
            }
          }
        }
      ]
    }
  }
}
```

## 自定义mapping建议

1. 写入一条文档到es的临时索引中，获取es自动生成的mapping
2. 修改步骤1得到的 mapping，自定义相关设置
3. 使用步骤2的 mapping 创建市级所需索引

比如下面一条要存入es的JSON文档,是一条nginx的日志

- JOSN

```
DELETE test_index

PUT test_index/doc/1
{
  "referer": "-",
  "response_code": "200",
  "remote_ip": "171.221.139.157",
  "method": "POST",
  "user_name": "-",
  "http_version": "1.1",
  "body_sent": {
    "bytes": "0"
  },
  "url": "/analyzeVideo"
}

GET test_index/_mapping
```

现在想要将url设置为 `text` 类型, body_sent 中的 types 设置成整型，其他默认为 keyword

做法就是先将文档在一个临时的索引中执行一下，因为es会自动的创建mapping

- mapping

```
{
  "test_index": {
    "mappings": {
      "doc": {
        "properties": {
          "body_sent": {
            "properties": {
              "bytes": {
                "type": "text",
                "fields": {
                  "keyword": {
                    "type": "keyword",
                    "ignore_above": 256
                  }
                }
              }
            }
          },
          "http_version": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "method": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "referer": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "remote_ip": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "response_code": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "url": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "user_name": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          }
        }
      }
    }
  }
}
```

接着将生成的 mapping 拷贝下来，放入到设置里面，这样所有的字段就都有了，只需要将不需要的配置删掉

- mapping

```
PUT my_product_index
{
  "mappings": {
    "doc": {
      "properties": {
        "body_sent": {
          "properties": {
            "bytes": {
              "type": "long"
            }
          }
        },
        "http_version": {
          "type": "keyword"
        },
        "method": {
          "type": "keyword"
        },
        "referer": {
          "type": "keyword"
        },
        "remote_ip": {
          "type": "keyword"
        },
        "response_code": {
          "type": "keyword"
        },
        "url": {
          "type": "text"
        },
        "user_name": {
          "type": "keyword"
        }
      }
    }
  }
}
```

创建完之后将文档放入到新创建的索引中

```
PUT my_product_index/doc/1
{
  "referer": "-",
  "response_code": "200",
  "remote_ip": "171.221.139.157",
  "method": "POST",
  "user_name": "-",
  "http_version": "1.1",
  "body_sent": {
    "bytes": "0"
  },
  "url": "/analyzeVideo"
}
```

如果不想一个字段一个字段去设置 keyword，就可以使用之前的 dynamic_template

```
PUT my_product_index
{
  "mappings": {
    "doc": {
      "dynamic_templates": [
        {
          "strings": {
            "match_mapping_type": "string",
            "mapping": {
              "type": "keyword"
            }
          }
        }
      ],
      "properties": {
        "body_sent": {
          "properties": {
            "bytes": {
              "type": "long"
            }
          }
        },
        "url": {
          "type": "text"
        }
      }
    }
  }
}
```

获取下 mapping

```
{
  "my_product_index": {
    "mappings": {
      "doc": {
        "dynamic_templates": [
          {
            "strings": {
              "match_mapping_type": "string",
              "mapping": {
                "type": "keyword"
              }
            }
          }
        ],
        "properties": {
          "body_sent": {
            "properties": {
              "bytes": {
                "type": "long"
              }
            }
          },
          "url": {
            "type": "text"
          }
        }
      }
    }
  }
}
```

## 索引模板

索引模板(Index Template), 主要用于在新建索引时自动应用预先设定的配置，简化索引创建的操作不愁

可以设定索引的配置和 mapping

可以有多个模板，根据 order 设置，order 大的覆盖小的配置

索引模板API, endpoint 为 _template

_template/test_template: template 名称

index_patterns: 匹配的索引名称

order: order 顺序配置

settings: 索引的配置

首先获取所有的 template

```
GET _template
```

创建两个模板

```
PUT _template/test_template
{
  "index_patterns": [
    "te*",
    "bar*"
  ],
  "order": 0,
  "settings": {
    "number_of_shards": 1
  },
  "mappings": {
    "doc": {
      "_source": {
        "enabled": false
      },
      "properties": {
        "name": {
          "type": "keyword"
        }
      }
    }
  }
}

PUT _template/test_template2
{
  "index_patterns": [
    "test*"
  ],
  "order": 1,
  "settings": {
    "number_of_shards": 1
  },
  "mappings": {
    "doc": {
      "_source": {
        "enabled": true
      }
    }
  }
}
```

接着再获取下模板

```
GET _template
```

创建索引并查看配置

```
PUT foo_index
GET foo_index/_mapping
```

上面是不匹配任何模板的，所以是空的

![](https://images.notes.xuepincat.com/elastic/elasticsearch/52.png)

创建匹配bar的test_template

```
PUT bar_index
GET bar_index/_mapping
```

![](https://images.notes.xuepincat.com/elastic/elasticsearch/53.png)

再创建匹配的test_idnex

```
DELETE test_index
PUT test_index
GET test_index
GET test_index/_mapping
```

![](https://images.notes.xuepincat.com/elastic/elasticsearch/54.png)

获取与删除API

```
GET _template
GET _template/test_template
DELETE _template/test_template
```
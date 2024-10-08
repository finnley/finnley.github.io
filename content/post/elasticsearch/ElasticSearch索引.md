+++
title = 'Elasticsearch索引'
date = 2019-04-11T16:22:55+08:00
draft = true
categories = [ "ElasticSearch" ]
tags = [ "es" ]
+++

创建
http://localhost:9200/es_video/1/
put

``` json
{
    "mappings": {
        "video": {
            "properties": { 
                "name": {
                    "type": "text"
                },
                "cat_id": {
                    "type": "integer"
                },
                "image": {
                    "type": "text"
                },
                "url": {
                    "type": "text"
                },
                "type": {
                    "type": "byte"
                },
                "content": {
                    "type": "text"
                },
                "uploader": {
                    "type": "keyword"
                },
                "create_time": {
                    "type": "integer"
                },
                "update_time": {
                    "type": "integer"
                },
                "status": {
                    "type": "byte"
                },
                "video_id": {
                    "type": "keyword"
                }
            }
        }
    }
}
```


新增
http://localhost:9200/es_video/video/1
put

``` json
{
        "name": "张学友1",
        "cat_id": 1,
        "image": "1.jpg",
        "url": "www.baidu.com",
        "type": 1,
        "uploader": "Finnley",
        "status": 1,
        "video_id": "123123"
}
``
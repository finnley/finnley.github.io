+++
title = '通过_analyze分析分词器standard和ik的区别'
date = 2019-04-01T05:35:15+08:00
draft = true
categories = [ "ElasticSearch" ]
tags = [ "es" ]
+++

## Windows 10安装 Curl
1. 工具下载

在官网处下载工具包： http://curl.haxx.se/download.html

<div align=center>
![](/images/elasticsearch/10.png)
</div>

2. 我将压缩包解压到了 `D:\Program Files` 这个目录下

<div align=center>
![](/images/elasticsearch/11.png)
</div>

3. 环境变量的配置，创建 `CURL_HOME` 的变量，将解压后的目录作为变量的值，然后保存

<div align=center>
![](/images/elasticsearch/12.png)
</div>

4. 找到 `PATH` 变量，追加下面内容

```
%CURL_HOME%\I386
```

<div align=center>
![](/images/elasticsearch/13.png)
</div>

5. `win+R` 打开命令窗口，输入指令 `curl --help` 进行测试

## standard

ES中默认的分词组件叫 `standard` ，ES是为我们提供RESTful API 的应用，Window我们可以通过下面命令进行测试

```
curl "http://192.168.1.100:9200"
```

<div align=center>
![](/images/elasticsearch/14.png)
</div>

我们可以通过下面指令进行测试分词组件

```
curl -XPOST "http://192.168.1.100:9200/_analyze?analyzer=standard&pretty" -d '这是一个商品的标题'
```

回车之后会发现提示下面错误

<div align=center>
![](/images/elasticsearch/15.png)
</div>

解决方法：

curl -H 'Content-Type: application/json' -XPOST "http://192.168.1.100:9200/_analyze?analyzer=standard&pretty" -d '这是一个商品的标题'


```
curl -H "Content-Type: application/json" -XPOST "http://192.168.1.100:9200/_analyze?analyzer=standard&pretty" -d '这是一个商品的标题'
```
curl -H "Content-Type: application/json" -XPOST "http://192.168.1.100:9200/_analyze?analyzer=standard&pretty" -d '{"content":"这是一个商品的标题"}'

curl -H "Content-Type: application/json;charset:utf-8;" -XPOST "http://192.168.1.100:9200/_analyze?analyzer=standard&pretty" -d '{"content":"这是一个商品的标题"}'


curl -XPOST http://192.168.1.100:9200/_analyze?analyzer=standard&pretty -H 'Content-Type:application/json' -d'
{"content":"美国留给伊拉克的是个烂摊子吗"}
'


curl -H "Content-Type: application/json" -XPOST http://192.168.1.100:9200/_analyze?analyzer=standard&pretty -d "{\"name\":\"Fred\"}"

{"analyzer": "ik_max_word", "text": "我是中国人"}
+++
title = 'codec 插件详解'
date = 2019-09-22T21:33:23+08:00
draft = true
categories = [ "ElasticSearch" ]
tags = [ "es" ]
+++

## Codec Plugin

Codec Plugin 作用于 input 和 output plugin , 负责将数据在原始与Logstash Event 之间转换，常见的 codec 有：

- plain 读取原始内容
- dots 将内容简化为点进行输出
- rubydebug 将Logstash Events 按照ruby 格式输出，方便调试
- line 处理带有换行符的内容
- json 处理json格式的内容
- multiline 处理多行数据的内容

输入是按行来读取数据，输出是rubydebug

```
bin/logstash -e "input {stdin {codec => line}} output {stdout {codec => rubydebug}}"
```

![](https://images.notes.xuepincat.com/elastic/logstash/6.png)

将输出设置为点

```
bin/logstash -e "input {stdin {codec => line}} output {stdout {codec => dots}}"
```

![](https://images.notes.xuepincat.com/elastic/logstash/7.png)

当我们不想看到详细的输出，只想看到一个进度的时候，logstash 是否在正常运行，比如在做压测的时候，一般会把输出设置成一个点，会看到屏幕在不断的打点

输入为json,输出是rubydebug

```
bin/logstash -e "input {stdin {codec => json}} output {stdout {codec => rubydebug}}"
```

比如先输入 hello,world，结果如下,发现报错了,加了个 tags

```
{
      "@version" => "1",
          "host" => "finnleys-MacBook-Pro.local",
          "tags" => [
        [0] "_jsonparsefailure"
    ],
    "@timestamp" => 2020-06-27T14:47:43.902Z,
       "message" => "hello,world"
}
```

![](https://images.notes.xuepincat.com/elastic/logstash/43.png)

如果输入一个正常的 {"name": "elk"},就没有报错，还将 name 解析出来了,message 也没有了，原始数据没有返回，是直接将json内容读取到字段里面

```
{"name": "elk"}
{
          "host" => "finnleys-MacBook-Pro.local",
          "name" => "elk",
    "@timestamp" => 2020-06-27T14:50:33.908Z,
      "@version" => "1"
}
```

```
  {"name": "elk","age": 50}
{
          "host" => "finnleys-MacBook-Pro.local",
          "name" => "elk",
           "age" => 50,
    "@timestamp" => 2020-06-27T14:52:43.628Z,
      "@version" => "1"
}
```

![](https://images.notes.xuepincat.com/elastic/logstash/44.png)

### multiline

当一个Event 的message 由多行组成，需要使用 codec， 常见的情况是堆栈日志信息的处理，如下所示：

```
Exception in thread "main" java.lang.NUllPointerException
    at com.examplt.myproject.Book.getTitle(Book.java:16)
    at com.example.myproject.Author.getBookTitles(Author.java:25)
    at com.example.myproject.Bookstrap.main(Bootstrap.java:14)
```

主要设置参数如下：

* pattern 设置行匹配的正则表达式，可以使用 grok
* what previous|next, 如果匹配成功，那么匹配的行是归属上一个事件还是下一个事件
* negate true or false 是否对 pattern 的结果取反

#### 实例1

```
Exception in thread "main" java.lang.NUllPointerException
    at com.example.myproject.Book.getTitle(Book.java:16)
    at com.example.myproject.Author.getBookTitles(Author.java:25)
    at com.example.myproject.Bootstrap.main(Bootstrap.java:14)
```

```
input {
    stdin {
        codec => multiline {
            pattern => "^\s"
            what => "previous"
        }
    }
}

output {
    stdout {
        codec => rubydebug
    }
}
```

java 堆栈的信息前面有空格，这些有空格的都属于第一行的内容，所以 pattern 需要写成以空格开头，
    `^\s` 表示以空格开头，what 为 previous,表示属于前面

conf/codec-multiline1.conf 配置文件

```
input {
    stdin {
        codec => multiline {
            pattern => "^\s"
            what => "previous"
        }
    }
}

output {
    stdout {
        codec => rubydebug
    }
}
```

执行

```
bin/logstash -f conf/codec-multiline1.conf
```

然后将堆栈信息输入

![](https://images.notes.xuepincat.com/elastic/logstash/9.png)

当我输入堆栈信息的时候没有反应，是因为 logstash不知道下一行是不是也是空格，所以需要在输入一遍堆栈信息

#### 实例2

```
printf("%10.10ld \t %10.10ld \t %s\
%f", w, x, y, z);
```
第一行最后斜线表示换行，在第二行可以继续输入，下面是对于这种情况的匹配

```
input {
    stdin {
        codec => multiline {
            pattern => "\\$"
            what => "next"
        }
    }
}
```

    pattern => "\\$"
    what => "next"
    表示以斜线结尾的这一行与它们的下一行组成同一个 logstash event

conf/codec-multiline2.conf

```
input {
    stdin {
        codec => multiline {
            pattern => "\\$"
            what => "next"
        }
    }
}

output {
    stdout {
        codec => rubydebug
    }
}
```

运行 

```
bin/logstash -f conf/codec-multiline2.conf
```

将内容输入

![](https://images.notes.xuepincat.com/elastic/logstash/10.png)

#### 实例3

```
[2019-09-22 11:49:14,389][INFO  ][env           ][Letha] using [1] data paths,mounts [[/
(/dev/disk1)]]，net usable_space [34.5gb], net total_space [118.9gb], types [hfs]
```

```
input {
    stdin {
        codec => multiline {
            pattern => "^\[%{TIMESTAMP_ISO8601}\]"
            negate => true
            what => "previous"
        }
    }
}

output {
    stdout {
        codec => rubydebug
    }
}
```

    negate => true 表示不以 [时间戳]开头的行都属于上一个 logstash event 

运行 

```
bin/logstash -f conf/codec-multiline3.conf
```

将内容输入

![](https://images.notes.xuepincat.com/elastic/logstash/11.png)
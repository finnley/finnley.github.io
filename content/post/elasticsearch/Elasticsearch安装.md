+++
title = 'Elasticsearch安装'
date = 2020-05-17T22:04:34+08:00
draft = true
categories = [ "ElasticSearch" ]
tags = [ "es" ]
+++

# 安装

1.下载 [Elasticsearch](https://elasticsearch.cn/download/)


2.解压进入到 ES 目录，启动

```
$ bin/elasticsearch -Ecluster.routing.allocation.disk.threshold_enabled=false -Epath.data=es
```

![](https://images.notes.xuepincat.com/elastic/elasticsearch/1.png)

如果启动时提示下面错误：

```
Exception in thread "main" java.nio.file.AccessDeniedException: /opt/es/6.3/elasticsearch-6.3.2/config/jvm.options
	at sun.nio.fs.UnixException.translateToIOException(UnixException.java:84)
	at sun.nio.fs.UnixException.rethrowAsIOException(UnixException.java:102)
	at sun.nio.fs.UnixException.rethrowAsIOException(UnixException.java:107)
	at sun.nio.fs.UnixFileSystemProvider.newByteChannel(UnixFileSystemProvider.java:214)
	at java.nio.file.Files.newByteChannel(Files.java:361)
	at java.nio.file.Files.newByteChannel(Files.java:407)
	at java.nio.file.spi.FileSystemProvider.newInputStream(FileSystemProvider.java:384)
	at java.nio.file.Files.newInputStream(Files.java:152)
	at org.elasticsearch.tools.launchers.JvmOptionsParser.main(JvmOptionsParser.java:58)
```

可以使用下面解决方案，然后再启动：

```
$ sudo chown -R finnley elasticsearch-6.3.2
```

3.下载 [Kibana](https://elasticsearch.cn/download/)

4.解压进入到 Kibana 目录，启动

```
$ bin/kibana
```

{% img https://images.notes.xuepincat.com/elastic/elasticsearch/2.png 828 %}

启动成功后访问 `127.0.0.1:5601` 进行访问，下图就是 `Kibana` 的欢迎界面

![](https://images.notes.xuepincat.com/elastic/elasticsearch/3.png)
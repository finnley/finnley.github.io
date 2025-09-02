+++
title = 'ElasticSearch的安装以及中文分词IK的安装和配置'
date = 2019-04-01T04:21:26+08:00
draft = true
categories = [ "ElasticSearch" ]
tags = [ "es" ]
+++

## Windows 10

### 安装并简单配置

1. 进入官网进行下载 `https://www.elastic.co/cn/downloads/elasticsearch`

<div align=center>
![](/images/elasticsearch/1.png)
</div>

2. 解压到 `D:\Apache\ElasticSearch` 目录中

<div align=center>
![](/images/elasticsearch/2.png)
</div>

<div align=center>
![](/images/elasticsearch/3.png)
</div>

3. 进入到 `D:\Apache\ElasticSearch\elasticsearch-6.7.0\config` 目录中，打开 `elasticsearch.yml` 配置文件

4. 找到 `cluster.name` 的cluster集群进行配置

```
#cluster.name: my-application
```

去掉前面的 `#` ,并修改

```
cluster.name: yii2-search
```

5. 找到 `node.name` 的节点进行配置

```
#node.name: node-1
```

去掉前面的 `#` ,并修改

```
node.name: master-1
```

6. 找到 `network.host` 的主机IP进行配置

```
#network.host: 192.168.0.1
```

去掉前面的 `#` ,并修改

```
network.host: 192.168.1.100
```

`192.168.1.100` 也是我本地的主机地址

7. 找到 `http.port` 的端口进行配置

```
#http.port: 9200
```

去掉前面的 `#`

```
http.port: 9200
```

8. 保存

9. 启动

windows下启动只需要进入bin 目录，双击执行 `elasticsearch.bat`

<div align=center>
![](/images/elasticsearch/4.png)
</div>


10. 启动之后我们可以通过在浏览器中输入 `192.168.1.100:9200` (IP:端口)的方式在浏览器中查看是否生效

<div align=center>
![](/images/elasticsearch/5.png)
</div>

### 安装中文分词插件

我们可以在 `GitHub` 上去搜索 `ik` 中文分词插件 `https://github.com/medcl/elasticsearch-analysis-ik`

<div align=center>
![](/images/elasticsearch/6.png)
</div>

我们可以针对不同的 `ES` 版本下载不同的插件

1. 进入 `D:\Apache\ElasticSearch\elasticsearch-6.7.0\plugins` 目录中创建 `ik` 文件夹

2. 打开 `https://github.com/medcl/elasticsearch-analysis-ik/releases` 下载对应版本，我当前 `ES` 版本是 `6.7.0` ,所以我下载也是与之对应的版本

<div align=center>
![](/images/elasticsearch/8.png)
</div>


2.  进入 `D:\Apache\ElasticSearch\elasticsearch-6.7.0\plugins\ik` 目录，将下载的文件解压到该目录中

<div align=center>
![](/images/elasticsearch/9.png)
</div>

4. 重启之后才会重新加载 `ik` 插件

## Linux(Ubuntu 18.04)

1. 进入官网进行下载 `https://www.elastic.co/cn/downloads/elasticsearch`

<div align=center>
![](/images/elasticsearch/1.png)
</div>

2.将安装包移动到 `/use/local` 目录下

```
sudo mv elasticsearch-6.7.1-linux-x86_64.tar.gz /usr/local/
```

3. 解压

```
sudo tar -zxvf elasticsearch-6.7.1-linux-x86_64.tar.gz 
```

4. 进入 `elasticsearch-6.7.1/config` 目录找到 `elasticsearch.yml` 进行配置

```
cd elasticsearch-6.7.1/config/
ls
sudo vim elasticsearch.yml  
```

在最下面添加下面配置内容

```
xpack.ml.enabled: false
#network.host: 0.0.0.0
http.port: 9200

#memory
bootstrap.memory_lock: false
bootstrap.system_call_filter: false

http.cors.enabled: true
http.cors.allow-origin: "*"
```

5. 保存后进入上一层 `bin` 目录中

```
cd ../bin
./elasticsearch
```

在 Ubuntu 上出现了没有权限的情况

```
ornnth@ornnth-Alienware-15-R3:/usr/local$ ./elasticsearch-6.7.1/bin/elasticsearch
Exception in thread "main" java.nio.file.AccessDeniedException: /usr/local/elasticsearch-6.7.1/config/jvm.options
	at sun.nio.fs.UnixException.translateToIOException(UnixException.java:84)
	at sun.nio.fs.UnixException.rethrowAsIOException(UnixException.java:102)
	at sun.nio.fs.UnixException.rethrowAsIOException(UnixException.java:107)
	at sun.nio.fs.UnixFileSystemProvider.newByteChannel(UnixFileSystemProvider.java:214)
	at java.nio.file.Files.newByteChannel(Files.java:361)
	at java.nio.file.Files.newByteChannel(Files.java:407)
	at java.nio.file.spi.FileSystemProvider.newInputStream(FileSystemProvider.java:384)
	at java.nio.file.Files.newInputStream(Files.java:152)
	at org.elasticsearch.tools.launchers.JvmOptionsParser.main(JvmOptionsParser.java:60)
```

解决方法

```
sudo chown -R ornnth:ornnth elasticsearch-6.7.1/
```

然后再去执行

```
./elasticsearch-6.7.1/bin/elasticsearch
```

6. 打开浏览器输入 `localhost:9200`

<div align=center>
![](/images/elasticsearch/16.png)
</div>

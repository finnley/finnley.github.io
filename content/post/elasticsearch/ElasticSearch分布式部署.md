+++
title = 'Elasticsearch分布式部署'
date = 2019-04-11T15:38:29+08:00
draft = true
categories = [ "ElasticSearch" ]
tags = [ "es" ]
+++


1. 配置

```
cd elasticsearch-6.7.1/config/
ls
sudo vim elasticsearch.yml 
```

添加下面内容到最下面,配置集群名称，节点名字以及是否作为主节点

```
cluster.name: master-1
node.name: node-1
node.master: true
```

然后启动服务

```
cd ../bin
./elasticsearch -d
```

在浏览器中输入 `localhost:9200` 进行查看

<div align=center>
![](/images/elasticsearch/19.png)
</div>

2. 创建多节点

在 `/user/local` 目录下创建 es2和es3目录

```
cd /usr/local
sudo mkdir es2 es3
sudo cp elasticsearch-6.7.1-linux-x86_64.tar.gz es2/
sudo cp elasticsearch-6.7.1-linux-x86_64.tar.gz es3/
```

3. 进入到 es2 目录进行加压

```
sudo tar -zxvf elasticsearch-6.7.1-linux-x86_64.tar.gz 
sudo chown -R ornnth:ornnth elasticsearch-6.7.1
```

4. 将 `/usr/local/elasticsearch-7.0/config` 目录中的 `elasticsearch.yml` 配置文件复制到 `es2` 中的目录中（此时的我所在的目录是在es2中）

```
sudo cp ../elasticsearch-6.7.1/config/elasticsearch.yml elasticsearch-6.7.1/config/
```

5. 修改配置

```
sudo gedit elasticsearch-6.7.1/config/elasticsearch.yml 
```

修改内容如下：

```
xpack.ml.enabled: false
#network.host: 192.168.1.100
http.port: 9202

#memory
bootstrap.memory_lock: false
bootstrap.system_call_filter: false

http.cors.enabled: true
http.cors.allow-origin: "*"

cluster.name: master-1
node.name: node-2
node.master: false
discovery.zen.ping.unicast.hosts: ["127.0.0.1"]
```

6. 保存退出并启动

```
./elasticsearch-6.7.1/bin/elasticsearch
```

在浏览器冲输入 `http://localhost:9202/` 进行查看

<div align=center>
![](/images/elasticsearch/18.png)
</div>

此时在浏览器中输入 `localhost:9100` 使用head插件查看

<div align=center>
![](/images/elasticsearch/20.png)
</div>

星号表示master主节点, 圆球表示从节点

7. 进入 `es3` 目录中解压

```
cd /usr/local/es3
sudo tar -zxvf elasticsearch-6.7.1-linux-x86_64.tar.gz 
sudo chown -R ornnth:ornnth elasticsearch-6.7.1
```

8. 将es2中的配置文件复制过来（此时我实在es3中）

```
sudo cp ../es2/elasticsearch-6.7.1/config/elasticsearch.yml ./elasticsearch-6.7.1/config/
```

9. 修改配置

```
sudo gedit elasticsearch-6.7.1/config/elasticsearch.yml 
```

修改内容如下：

```
xpack.ml.enabled: false
#network.host: 192.168.1.100
http.port: 9203

#memory
bootstrap.memory_lock: false
bootstrap.system_call_filter: false

http.cors.enabled: true
http.cors.allow-origin: "*"

cluster.name: master-1
node.name: node-3
node.master: false
discovery.zen.ping.unicast.hosts: ["127.0.0.1"]
```

10. 保存退出并启动

```
./elasticsearch-6.7.1/bin/elasticsearch
```

在浏览器冲输入 `http://localhost:9203/` 进行查看

<div align=center>
![](/images/elasticsearch/21.png)
</div>

此时在浏览器中输入 `localhost:9100` 使用head插件查看

<div align=center>
![](/images/elasticsearch/22.png)
</div>

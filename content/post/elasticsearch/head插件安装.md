+++
title = 'head插件安装'
date = 2019-04-11T14:11:08+08:00
draft = true
categories = [ "ElasticSearch" ]
tags = [ "es" ]
+++

1. 进入Github进行下载 `https://github.com/mobz/elasticsearch-head`

```
cd /usr/local
sudo git clone git://github.com/mobz/elasticsearch-head.git
cd elasticsearch-head
npm install
npm run start
```

2. 开启 es 服务

```
./elasticsearch -d
```

3. 在浏览器输入 `http://localhost:9100`

<div align=center>
![](/images/elasticsearch/17.png)
</div>
+++
title = 'MySQL知识库'
date = 2024-07-29T11:08:33+08:00
draft = true
categories = [ "MySQL" ]
tags = [ "mysql" ]
+++


# mysql.mysql 表Host取值以及作用

<img class="gkpho-img" style="width:240px;" src="/img/mysql/knowledge/10.jpg">

mysql.user 中`Host` 字段表示允许连接到数据库的主机地址，Host 的值可能值以及作用如下：

- `%`： 表示允许任意主机通过网络连接到数据库；
- `127.0.0.1`：表示只允许本地主机（即当前运行 MySQL 服务器的主机）通过本地网络连接到数据库；
- `localhost`：也表示只允许本地主机连接，但本质上是通过socket方式连接，推荐的连接方式
- `具体IP地址`：则是指具体的某个IP地址，比如 `192.168.1.100`，这表示只允许该IP地址的主机连接到数据库。设置具体的IP地址可以限制连接的主机范围，增强数据库的安全性。
+++
title = 'MySQL查询'
date = 2024-05-03T18:13:37+08:00
draft = false
categories = [ "MySQL" ]
tags = [ "mysql" ]
+++

## 示例数据库 Sakila-db 安装

Sakila-db 示例数据库比较知名，因为它在很多教学案例中都扮演了很重要的角色。

* Sakila 是 MySQL中的一个实例数据库（sample database）
* Sakila 展示了一个电影DVD租赁公司的后台管理系统的数据库
* 很多国外教程都使用了 Sakila 作为案例

下载地址：

MySQL官网-> DOCUMENTATION -> More -> sakila database

![](/img/mysql/250.png)

```
https://downloads.mysql.com/docs/sakila-db.tar.gz
https://downloads.mysql.com/docs/sakila-db.zip
```

下载解压之后有个sakila目录，里面有3个文件：

```
sakila-data.sql
sakila-schema.sql
sakila.mwb
```

schema 是数据结构，data 是数据文件，mwb 是MySQL数据结构的可视化文件

安装过程如下：

```shell
# mysql -uroot -p123123
```

```sql
source /root/sakila-db/sakila-schema.sql
source /root/sakila-db/sakila-data.sql
```

查看是否成功导入

```sql
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sakila             |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

mysql> use sakila
Database changed
mysql> show tables;
+----------------------------+
| Tables_in_sakila           |
+----------------------------+
| actor                      |
| actor_info                 |
| address                    |
| category                   |
| city                       |
| country                    |
| customer                   |
| customer_list              |
| film                       |
| film_actor                 |
| film_category              |
| film_list                  |
| film_text                  |
| inventory                  |
| language                   |
| nicer_but_slower_film_list |
| payment                    |
| rental                     |
| sales_by_film_category     |
| sales_by_store             |
| staff                      |
| staff_list                 |
| store                      |
+----------------------------+
23 rows in set (0.00 sec)
```
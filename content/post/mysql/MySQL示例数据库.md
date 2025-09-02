+++
title = 'MySQL示例数据库'
date = 2024-04-03T18:13:37+08:00
draft = false
categories = [ "MySQL" ]
tags = [ "mysql" ]
+++

# 1 sakila database

sakila database 示例数据库比较知名，因为它在很多教学案例中都扮演了很重要的角色。

* Sakila 是 MySQL中的一个示例数据库（sample database）
* Sakila 展示了一个电影DVD租赁公司的后台管理系统的数据库
* 很多国外教程都有在使用 Sakila 作为案例

## 1.2 下载

【MySQL官网】->【DOCUMENTATION】->【More】->【sakila database】

![](/img/mysql/250.png)

```bash
wget https://downloads.mysql.com/docs/sakila-db.tar.gz
```

下载解压后目录中有3个文件：

```bash
sakila-data.sql
sakila-schema.sql
sakila.mwb
```

- schema：数据结构文件
- data：数据文件
- mwb：MySQL数据结构的可视化文件

## 1.2 安装

```shell
# mysql -uroot -p123123
```

```sql
source /root/sakila-db/sakila-schema.sql
source /root/sakila-db/sakila-data.sql
```

## 1.3 验证

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

## 1.4 小结

登录MySQL 并导入结构以及数据文件：
```sql
source /root/sakila-db/sakila-schema.sql
source /root/sakila-db/sakila-data.sql
```

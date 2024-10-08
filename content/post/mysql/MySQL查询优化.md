+++
title = 'MySQL查询优化'
date = 2024-05-04T07:35:23+08:00
draft = true
categories = [ "MySQL" ]
tags = [ "mysql" ]
+++

## 覆盖索引如何提高查询效率

什么是覆盖索引？

查询语句从执行到返回结果均使用同一个索引。覆盖索引可以有效减少回表。

实验-验证联合索引的作用：

复制 sakila 库中的 inventory 存货数据表，去掉外键、联合索引：

原表：

```sql
DROP TABLE IF EXISTS `inventory`;
CREATE TABLE `inventory` (
  `inventory_id` mediumint(8) unsigned NOT NULL AUTO_INCREMENT,
  `film_id` smallint(5) unsigned NOT NULL,
  `store_id` tinyint(3) unsigned NOT NULL,
  `last_update` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`inventory_id`),
  KEY `idx_fk_film_id` (`film_id`),
  KEY `idx_store_id_film_id` (`store_id`,`film_id`),
  CONSTRAINT `fk_inventory_film` FOREIGN KEY (`film_id`) REFERENCES `film` (`film_id`) ON UPDATE CASCADE,
  CONSTRAINT `fk_inventory_store` FOREIGN KEY (`store_id`) REFERENCES `store` (`store_id`) ON UPDATE CASCADE
) ENGINE=InnoDB AUTO_INCREMENT=4582 DEFAULT CHARSET=utf8mb4;
```

复制表：

```sql
CREATE TABLE `inventory_1` (
  `inventory_id` mediumint(8) unsigned NOT NULL AUTO_INCREMENT,
  `film_id` smallint(5) unsigned NOT NULL,
  `store_id` tinyint(3) unsigned NOT NULL,
  `last_update` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`inventory_id`),
  KEY `idx_fk_film_id` (`film_id`)
) ENGINE=InnoDB AUTO_INCREMENT=4582 DEFAULT CHARSET=utf8mb4;
```

导入数据：

```sql
INSERT INTO inventory_1 SELECT * FROM inventory;
```

查询以下语句的执行计划：

```sql
explain select store_id, film_id from sakila.`inventory_1` where store_id = 1;
explain select store_id, film_id from sakila.`inventory` where store_id = 1;
```

```sql
mysql> explain select store_id, film_id from sakila.`inventory_1` where store_id = 1;
+----+-------------+-------------+------------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table       | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------------+------------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | inventory_1 | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 4581 |    10.00 | Using where |
+----+-------------+-------------+------------+------+---------------+------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)
```

第一条语句 `possible_keys` 列表示这条SQL执行可能用到哪些索引，值为 NULL 表示没有用到任何索引；<br>
`key` 列表示命中了哪些索引，值为 NULL 表示没有命中任何索引；<br>
`Extra` 列表示 `Using where` 扫描

综上，第一条语句没有走任何索引，走的是全表扫描。

```sql
mysql> explain select store_id, film_id from sakila.`inventory` where store_id = 1;
+----+-------------+-----------+------------+------+----------------------+----------------------+---------+-------+------+----------+-------------+
| id | select_type | table     | partitions | type | possible_keys        | key                  | key_len | ref   | rows | filtered | Extra       |
+----+-------------+-----------+------------+------+----------------------+----------------------+---------+-------+------+----------+-------------+
|  1 | SIMPLE      | inventory | NULL       | ref  | idx_store_id_film_id | idx_store_id_film_id | 1       | const | 2270 |   100.00 | Using index |
+----+-------------+-----------+------------+------+----------------------+----------------------+---------+-------+------+----------+-------------+
1 row in set, 1 warning (0.01 sec)
```

第二条语句 `possible_keys` 和 `key` 列告诉我们执行这条SQL走的是 `idx_store_id_film_id` 索引，`Extra` 列值为 `Using index`，`Using index` 在查询计划里面就叫 `索引覆盖`。也就是使用了同一条索引，同一条索引不需要回表。使用同一个索引就可以完成从数据的搜索到数据结果的返回。

查看两张表的区别：

```sql
mysql> show index from inventory;
+-----------+------------+----------------------+--------------+--------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| Table     | Non_unique | Key_name             | Seq_in_index | Column_name  | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment |
+-----------+------------+----------------------+--------------+--------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| inventory |          0 | PRIMARY              |            1 | inventory_id | A         |        4581 |     NULL | NULL   |      | BTREE      |         |               |
| inventory |          1 | idx_fk_film_id       |            1 | film_id      | A         |         958 |     NULL | NULL   |      | BTREE      |         |               |
| inventory |          1 | idx_store_id_film_id |            1 | store_id     | A         |           2 |     NULL | NULL   |      | BTREE      |         |               |
| inventory |          1 | idx_store_id_film_id |            2 | film_id      | A         |        1521 |     NULL | NULL   |      | BTREE      |         |               |
+-----------+------------+----------------------+--------------+--------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
4 rows in set (0.00 sec)

mysql> show index from inventory_1;
+-------------+------------+----------------+--------------+--------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| Table       | Non_unique | Key_name       | Seq_in_index | Column_name  | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment |
+-------------+------------+----------------+--------------+--------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| inventory_1 |          0 | PRIMARY        |            1 | inventory_id | A         |        4581 |     NULL | NULL   |      | BTREE      |         |               |
| inventory_1 |          1 | idx_fk_film_id |            1 | film_id      | A         |        4581 |     NULL | NULL   |      | BTREE      |         |               |
+-------------+------------+----------------+--------------+--------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
2 rows in set (0.01 sec)
```

区别在于原表 `store_id` 和 `film_id` 两个字段组成的联合索引，其中 store_id 在联合索引中排第一位（Seq_in_index 值为 1），film_id 在联合索引中排第二位（Seq_in_index 值为 2），新建的复制表没有联合索引。

再看下查询的SQL `select store_id, film_id from sakila.`inventory` where store_id = 1;` 首先判断SQL是否可以使用联合索引，回答是可以使用的，是因为联合索引中有 store_id，where 查询中就是以 store_id 作为条件的，所以数据查询时是会走索引的。

详细介绍还需要细品。

实验2：

```sql
explain select inventory_id, store_id, film_id from sakila.`inventory` where store_id = 1;
explain select inventory_id, store_id, film_id, last_update from sakila.`inventory` where store_id = 1;
``` 

下面SQL查询字段中又添加了一个 `last_update` 字段

执行计划如下：

```sql
mysql> explain select inventory_id, store_id, film_id from sakila.`inventory` where store_id = 1;
+----+-------------+-----------+------------+------+----------------------+----------------------+---------+-------+------+----------+-------------+
| id | select_type | table     | partitions | type | possible_keys        | key                  | key_len | ref   | rows | filtered | Extra       |
+----+-------------+-----------+------------+------+----------------------+----------------------+---------+-------+------+----------+-------------+
|  1 | SIMPLE      | inventory | NULL       | ref  | idx_store_id_film_id | idx_store_id_film_id | 1       | const | 2270 |   100.00 | Using index |
+----+-------------+-----------+------------+------+----------------------+----------------------+---------+-------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)

```

首先第一条SQL是可以命中索引的，因为 `store_id = 1` 是联合索引的带头字段，索引它是可以走 `store_id` 和 `film_id` 联合索引去查询，那为什么 `inventory_id, store_id, film_id` 查询这三个字段可以走索引覆盖，不需要回表呢？<br>
之前的SQL查询返回的两个字段 `store_id, film_id` 是联合索引的两个字段，联合索引里面已经包含了这两个字段，所以不用回表再查。那为什么这条SQL语句多了个 `inventory_id` 也不需要回表呢？<br>
因为 `inventory_id` 是主键，也就是说联合索引不光有 `store_id` 和 `film_id` 排序字段，还有主键，主键本来是用来回表用的，`store_id` 和 `film_id` 查询出来之后查到的是主键，然后再用主键去回表查询其他的字段，但是现在MySQL发现用 `store_id` 查询出来之后要返回的字段是 `store_id` 和 `film_id` 全部包含在一条索引里面，所以还是能走索引

```sql
mysql> explain select inventory_id, store_id, film_id, last_update from sakila.`inventory` where store_id = 1;
+----+-------------+-----------+------------+------+----------------------+----------------------+---------+-------+------+----------+-------+
| id | select_type | table     | partitions | type | possible_keys        | key                  | key_len | ref   | rows | filtered | Extra |
+----+-------------+-----------+------------+------+----------------------+----------------------+---------+-------+------+----------+-------+
|  1 | SIMPLE      | inventory | NULL       | ref  | idx_store_id_film_id | idx_store_id_film_id | 1       | const | 2270 |   100.00 | NULL  |
+----+-------------+-----------+------------+------+----------------------+----------------------+---------+-------+------+----------+-------+
1 row in set, 1 warning (0.00 sec)
```

第二条SQL，就多查询了 last_update 字段，发现使用了 `idx_store_id_film_id` 索引，这是因为查询条件是 `store_id = 1` 是联合索引的左边的字段，可以使用联合索引去搜索符合 where 条件的的数据记录，但是 Extra 的值是 NULL，说明没有走索引覆盖，而是回表了，也就是回到主索引去搜索数据了，为什么呢？这是因为多返回了 last_update 字段，多返回的 last_update 字段在 idx_store_id_film_id 索引中是没有的，由于索引idx_store_id_film_id中没有 last_update的信息，所以需要拿inventory_id主键回去再索引，把整行取出来将 last_update 解析出来。所以这条语句一定是浪费时间的。

### 总结

* 覆盖索引通过取消回表操作来提升查询效率
* 若数据的查询不只使用了一个索引，则不是覆盖索引
* 可以通过优化SQL语句或优化联合索引，来使用覆盖索引


                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           
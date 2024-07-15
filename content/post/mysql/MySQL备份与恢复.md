+++
title = 'MySQL备份与恢复'
date = 2024-05-04T09:43:43+08:00
draft = false
categories = [ "MySQL" ]
tags = [ "mysql" ]
+++

## 概述

对任何数据库来说，备份都非常重要，数据库备份可以在关键时刻保证我们不丢失或者少丢失数据，虽然可以通过复制技术保留数据库的多个副本， 但数据库复制并不能取代备份的作用。

比如在主数据库上因为误操作删除了一批数据，由于主从复制时间很短，在我们发现时从库上的数据也可能被删除了，因此不能使用从库的数据来恢复主库上的数据，而应该使用原来的备份进行误删除数据的恢复。

## 备份分类（备份方式的分类维度）

### 备份时数据库的状态

也就是备份时数据库是锁了，别人都不能用，还是没锁，别人都可以用的情况下动态的备份。

#### 热备（Hot Backup）

备份时数据库正常运行中，也就是别人可以正常读写，这种方式备份对业务侵入最小 。（正常运行中直接备份）

#### 冷备（Cold Backup）

数据库完全停止后备份，就是将数据库停机，对业务侵入最大，业务既不能读也不能写，现在冷备基本都不用了。（完全停止后备份）

#### 温备（Warm Backup）

处于冷备和热备之间，数据库停止了，但没有完全停止，也就是数据库只能读，但是数据库不能再更新了，因为备份要从数据库拉数据，整个数据库不能动。（数据库只读）

### 备份文件的格式

备份的文件是以什么格式编排的。


#### 逻辑备份

输出文本或SQL语句，将数据按照逻辑的方式导出成人可读的逻辑性文本。

#### 物理备份（裸文件备份）

备份数据库原始底层文件，如果是 innoDB ，备份的就是 ibd 文件（ibd表空间）这些文件人不可读，但是这些文件里面是含有数据的。

### 备份内容

全备or增备

#### 全备份

备份完整的数据

#### 增量备份

备份这一次与上一次的数据差异，比如数据增加的行，减少的行，修改的行。

#### 日志备份

备份 binlog 日志，binlog 表示的是数据库在这一段时间所有数据差异和变化的操作，日志备份其实也是增量备份的一种。

增量备份一般是逻辑的，日志备份正常是将binlog文件备份下来，有点类似物理的增量备份。

## 工具

* mysqldump: 逻辑，热，全备工具，备份出来的是人可读的sql文件
* xtrabackup: 物理，热，全量+增量备份

## 最简单的备份（OUTFILE 命令备份）

* MySQL原生SQL指令
* 最原始的逻辑备份方式
* 备份的功能和效果取决于如何写SQL语句

1、首先查询MySQL的导出路径
 
mysql 都有个能够安全操作的文件目录，比如想把outfile内容导出成文件，就可以看下安装的MySQL server可操作的哪些系统文件路径

```sql
mysql> show variables like '%secure%';
+--------------------------+-----------------------+
| Variable_name            | Value                 |
+--------------------------+-----------------------+
| require_secure_transport | OFF                   |
| secure_auth              | ON                    |
| secure_file_priv         | /var/lib/mysql-files/ |
+--------------------------+-----------------------+
3 rows in set (0.00 sec)
```

![](/img/mysql/backup/10.png)

也就是导出时要导出到 /var/lib/mysql-files/ 这个文件夹里，这个文件夹才是mysqld 能操作的安全文件夹 

2、准备数据

```sql
mysql> create table t1 (f1 int, f2 int);
mysql> insert into t1 values(4, 6), (6, 3), (7, 1);
mysql> select * from t1;
+------+------+
| f1   | f2   |
+------+------+
|    4 |    6 |
|    6 |    3 |
|    7 |    1 |
+------+------+
```


3、使用 `into outfile` 指令将查询结果导出至文件

```sql
select * into outfile '/var/lib/mysql-files/out_file_test' from Z;
```

如：

```sql
mysql> select * into outfile '/var/lib/mysql-files/t1-out' from t1;
Query OK, 3 rows affected (0.02 sec)

mysql>
mysql> exit
Bye
[root@localhost sakila-db]# cat /var/lib/mysql-files/t1-out
4	6
6	3
7	1
[root@localhost sakila-db]#
```

也可以如下操作记录分隔符：

```sql
select * into outfile '/var/lib/mysql-files/t1-out2' fields terminated by ',' from t1;
mysql> exit
Bye
[root@localhost sakila-db]# cat /var/lib/mysql-files/t1-out2
4,6
6,3
7,1
[root@localhost sakila-db]#
```

现在有个场景：
我想备份所有的表，但是在备份 t1 这张表的时候 t2 表数据改动了，我原本是想要个一致性的视图，也及时从我备份开始的时候不要有任何数据库的改动，即备份同一时刻所有表的内容，这个outfile 是否可以做到呢？
可以
在InnoDB 事务下，可以做到一致性视图


### 存在问题

* 输出文本简略
* 输出的文件无法还原，往往用来简单导出数据

## mysqldump 备份

* 非常常用的MySQL逻辑备份工具
* MySQL Server 自带
* 输出的备份内容为 SQL语句，平衡了阅读和还原
* SQL 语句占用空间较小

### mysqldump 原理

粗糙些说就是通过往MySQL下发SQL指令，将所有表数据查询出来导入到SQL文件中。但是并不是下发的简单的SELECT语句。

* mysqldump 使用以下语句对数据进行备份：

```sql
SELECT SQL_NO_CACHE FROM `T`;
```

MySQL 8.0 之前的SQL语句结果都是有缓存的，主要是方便下次查询SQL语句的时候可以直接输出查询结果，不用到硬盘上找。但是这个特性对 mysqldump 是不友好的，mysqldump 会对每个表的结果作缓存（对每个表做select *），这样对别的用户查询时没有意义，还白白占用了 MySQL Server 层的缓存空间，所以需要这个命令对MySQL结果不进行缓存。

所以 SQL_NO_CACHE 查询出的数据不会进入SQL缓存

### 如何使用

直接导出的SQL文件即可进行还原

```sql
source test.sql
```

1、准备数据：

```sql
create database d1;
use d1;
create table z(a int, b int, c int, d int);
insert into z values(1,2,3,4),(5, 6, 7,8),(9, 10,11,23);
select * from z;
+------+------+------+------+
| a    | b    | c    | d    |
+------+------+------+------+
|    1 |    2 |    3 |    4 |
|    5 |    6 |    7 |    8 |
|    9 |   10 |   11 |   23 |
+------+------+------+------+
3 rows in set (0.01 sec)
```

2、mysqldump 使用以下语句对数据进行备份：

```shell
# mysqldump -uroot -p123123 --databases d1 --single-transaction > test.sql
mysqldump: [Warning] Using a password on the command line interface can be insecure.
```

`--single-transaction` 表示可重复读隔离级别进行备份，这样可以保证从备份第一个表到备份最后一个表，所备份的内容都是同一时刻的。


3、查看备份内容

```shell
# cat test.sql
-- MySQL dump 10.13  Distrib 5.7.44, for Linux (x86_64)
--
-- Host: localhost    Database: d1
-- ------------------------------------------------------
-- Server version	5.7.44

/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
/*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
/*!40101 SET NAMES utf8 */;
/*!40103 SET @OLD_TIME_ZONE=@@TIME_ZONE */;
/*!40103 SET TIME_ZONE='+00:00' */;
/*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;

--
-- Current Database: `d1`
--

CREATE DATABASE /*!32312 IF NOT EXISTS*/ `d1` /*!40100 DEFAULT CHARACTER SET latin1 */;

USE `d1`;

--
-- Table structure for table `z`
--

DROP TABLE IF EXISTS `z`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `z` (
  `a` int(11) DEFAULT NULL,
  `b` int(11) DEFAULT NULL,
  `c` int(11) DEFAULT NULL,
  `d` int(11) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=latin1;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `z`
--

LOCK TABLES `z` WRITE;
/*!40000 ALTER TABLE `z` DISABLE KEYS */;
INSERT INTO `z` VALUES (1,2,3,4),(5,6,7,8),(9,10,11,23);
/*!40000 ALTER TABLE `z` ENABLE KEYS */;
UNLOCK TABLES;
/*!40103 SET TIME_ZONE=@OLD_TIME_ZONE */;

/*!40101 SET SQL_MODE=@OLD_SQL_MODE */;
/*!40014 SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS */;
/*!40014 SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS */;
/*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
/*!40101 SET CHARACTER_SET_RESULTS=@OLD_CHARACTER_SET_RESULTS */;
/*!40101 SET COLLATION_CONNECTION=@OLD_COLLATION_CONNECTION */;
/*!40111 SET SQL_NOTES=@OLD_SQL_NOTES */;

-- Dump completed on 2024-05-04  3:05:14
```

从备份内容中看到 “CREATE TABLE `z` ...” 自动写了建表语句；还有 “INSERT INTO `z` VALUES (1,2,3,4),(5,6,7,8),(9,10,11,23);” 插入了三行数据

不光建库建表语句有，还有锁表的语句 “LOCK TABLES `z` WRITE; ... UNLOCK TABLES;” 这个主要用在还原上，因为需要保证数据一致性，故需要锁表并在还原后释放。

抓住了 outfile的痛点

### 注意事项

--single-transaction： 在 RR 级别下进行（InnoDB），保证第一张表到最后一张表都是同一时刻的数据，保证备份的一致性。
--lock-all-tables: 使用 FWWRL 锁所有表（MyISAM），FWWRL是个全局锁，所有表只能读不能写
--local-tables: 使用 READ LOCAL 锁当前库的表（MyISAM），如果允许牺牲一致性，允许第一张表到最后一张表时数据是有变化的，也就是备份到哪张表时就用 local-tables 锁住那张表
--all-databases: 备份所有苦
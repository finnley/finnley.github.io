+++
title = 'MySQL日志'
date = 2024-05-04T16:04:56+08:00
draft = false
categories = [ "MySQL" ]
tags = [ "mysql" ]
+++

## 日志分类

* error log 错误日志，用于排错，比如 /var/log/mysqld.log，该日志默认开启
* binlog 二进制日志，用于备份、增备、DDL、DML、DCL 等操作
* relay log 中继日志，用于复制，比如接收 replication master
* slow log 慢查询日志，用于调优，比如查询时间超过指定值的SQL语句

参考：https://dev.mysql.com/doc/refman/8.0/en/server-logs.html

## 日志的产生

* 数据库更新时，会产生 binlog, redo log, undo log
* binlog: server 层产生的逻辑日志，也就是它与 InnoDB, MyISAM等存储引擎是没有任何关系的。当数据库发生任何的增，删，改操作，除了查询外有任何数据库的变化，都会记录 binlog, binlog相当于数据库的更新记录。但这个日志不是给人看的，它是数据库运行逻辑上所必须的一种日志
* redo log: InngoDB 产生的物理日志，它是保证持久化。事务有ACID特点，其中重要的一个就是持久化（D）
* undo log: InnoDB 产生的逻辑日志，保证ACID中的隔离性（A）、原子性(I)。

## MySQL 日志体系

* MySQL为了满足主从复制、事务等，具有复杂的日志体系，这些日志不是给人看的，而是系统运行所需要的日志
* binlog: server 层产生的逻辑日志，与存储引擎无关，用于数据复制，用来记录系统数据更新的情况的日志。也就是 binlog 传送给从机或者备库，备库会用 binlog 来重现主库的数据更新。

它与 InnoDB, MyISAM等存储引擎是没有任何关系的。当数据库发生任何的增，删，改操作，除了查询外有任何数据库的变化，都会记录 binlog, binlog相当于数据库的更新记录。但这个日志不是给人看的，它是数据库运行逻辑上所必须的一种日志

* InnoDB产生的 undo log，redo log, 用来实现事务 ACID，与其他存储引擎无关，也和server层无关，是 InnoDB 特有的
* MySQL 的日志体系主要不是用来看的，而是运行必要的

### binlog 归档日志

binog 是 server 层产生的逻辑日志。逻辑日志是指记录的数据应该怎么变化，并不是记录数据的物理页具体怎么变（不懂）。

用来进行数据的复制和数据传送；有些时候用来记录数据的备份和数据的闪回，因为 binlog 完整记录了数据库每次的数据操作，可理解为数据库数据的审计，可作为数据闪回手段。
* Binlog 记录在专门的文件中

### undo log 回滚日志

* InnoDB 自身产生的逻辑日志，用于事务回滚和展示旧版本
* 对任何数据（缓存）的更新，都需要先写 undo log
* undo log 位于表空间的 undo segment 中
* undo log 记录数据的相反操作，如原来是 a, 更新为 b
    SQL: UPDATE NAME = 'b' -> undo: UPDATE name = 'a'

### redo log 重做日志

* InnoDB 自身产生的物理日志，记录数据页（B+树的子节点或叶子界面，里面记录的是数据本身）的变化
* InnoDB “日志优于数据”，记录 redo log 视为数据已经更新
* 内存中的数据更新后写 redo log，数据被写入硬盘后删除
* redo log 存储在4个1GB文件中，并且循环写入，1GB是可以配置的
  - write pos 是当前日志写入点
  - check point 是擦除点，数据被更新到硬盘时擦除
  - 当 write pos 追上 check point 时，事务无法提交（无法增删改，但是可以查），需要等待 check point 推进，强制刷 redo log 也是更新的时候数据库性能差的一个原因
  - 只要 redo log 不丢，数据就不会丢失，也就是已经落到文件上的 redo log
  - 在磁盘上对应的4个文件
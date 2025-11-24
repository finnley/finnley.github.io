+++
title = 'MySQL备份与恢复——outfile'
date = 2024-09-01T08:07:10
draft = true
categories = [ "MySQL" ]
tags = [ "mysql" ]
+++

# OUTFILE 命令备份

最简单的备份。

## 特点

* MySQL原生SQL指令
* 最原始的逻辑备份方式
* 备份的功能和效果取决于如何写SQL语句

## 使用

**数据准备**

参考 [MySQL示例数据库](https://notes.einscat.com/post/mysql/mysql%E7%A4%BA%E4%BE%8B%E6%95%B0%E6%8D%AE%E5%BA%93/)

**查询导出路径**
 
MySQL 都有个能够安全操作的文件目录，比如想把 outfile 内容导出成文件，就可以看下安装的 MySQL Server 可操作的系统文件路径

```sql
mysql> show variables like '%secure%';
+--------------------------+-----------------------+
| Variable_name            | Value                 |
+--------------------------+-----------------------+
| require_secure_transport | OFF                   |
| secure_auth              | ON                    |
| secure_file_priv         | /var/lib/mysql-files/ |
+--------------------------+-----------------------+
```

也就是导出时要导出到 `/var/lib/mysql-files/` 这个文件夹里，这个文件夹才是 MySQL 能操作的安全文件夹。 

**导出至文件**

将查询结果导出：
```sql
select * into outfile '/var/lib/mysql-files/out_file_test' from Z;
```

示例：
```sql
mysql> select * into outfile '/var/lib/mysql-files/store-out' from sakila.store;
Query OK, 3 rows affected (0.02 sec)

mysql> exit
Bye
# cat /var/lib/mysql-files/store-out
1	1	1	2006-02-15 04:57:12
2	2	2	2006-02-15 04:57:12
#
```

使用分隔符：

```sql
select * into outfile '/var/lib/mysql-files/store-out2' fields terminated by ',' from t1;
mysql> exit
Bye
# cat /var/lib/mysql-files/store-out2
1,1,1,2006-02-15 04:57:12
2,2,2,2006-02-15 04:57:12
#
```

## 小结

**归纳**

- OUTFILE 是最简单的的原生数据备份工具
- 使用简单、灵活，灵活指的是导出的取决于我们写的SQL
- 导出的数据无法还原，仅用来简单导出数据
- 在InnoDB事务下，可以做到一致性视图
- 修改分隔符：fields terminated by 
- 修改换行符：lines terminated by

 
# 思考

1、Outfile 是否可以备份整个数据库？针对不足之处如何改进？

需要select库的所有表，需要将所有的表数据查询一遍再导出。

这么操作很繁琐，如何改进？

希望可以做到自动发送 SELECT 语句，而不是手动发送。

希望可以做到自动开启事务。在支持事务的引擎下可以做到从备份第一张表到最后一张表，备份的都是同一时刻的数据。备份完之后可以自动提交事务。

希望可以支持 INSERT 语句，可直接用于还原。

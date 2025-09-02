+++
title = 'MySQL逻辑备份恢复工具——Dump Loading Utility'
date = 2024-09-01T08:07:10
draft = true
categories = [ "MySQL" ]
tags = [ "mysql" ]
+++

# 一、Dump Loading Utility

## （一）功能（逻辑转储和逻辑还原）

逻辑转储和逻辑还原就是我们常说的逻辑备份与恢复。

支持功能如下：

- 多线程备份
- 文件压缩
- 备份过程展示进度信息
- 备份文件包括 schema DDL 文件，包含数据以制表符分隔的 .tsv 文件
- 支持在本非期间锁定备份实例，以确保数据一致性，默认工具会将表数据分块到多个数据文件，并对文件进行压缩
- 实例备份不包含系统实例，schema备份不包含mysql表数据。

参考文档：[Dump Loading Utility](https://dev.mysql.com/doc/mysql-shell/8.0/en/mysql-shell-utilities-load-dump.html#mysql-shell-utilities-load-dump-requirements)


## （二）相关工具

名称                     | 作用
----                    | ----
util.dumpInstance()     | 备份数据库实例
util.dumpSchemas()      | 备份指定数据库 schema
util.dumpTables()       | 备份指定表或视图
util.loadDump()         | 恢复上述三个实用工具备份

## （三）环境依赖

由于该工具并非 MySQL 自带，需要额外安装，故需要考虑的点如下：

**操作系统**

几乎支持目前大多数主流操作系统。

**系统架构**

- ARM64
- x86 64bit
- x86 32bit

**glibc**

- glibc 2.16
- glibc 2.17
- glibc 2.28

**兼容性**

- 仅支持 MySQL Server GA 版，DMP 目前满足此条件。
- 支持 MySQL 5.7 以上版本，MySQL 5.6版本支持不完整。

**注意事项**

- 8.0.27 之前的 shell 工具无法恢复 8.0.27 版本之后备份出来的数据，所以建议使用最新版本（截止2024-09-05 23:20:11，最新版本为 8.0.38）
- 实例或模式中的对象名称必须使用 latin1 或 utf8 字符集。
- 只有使用 InnoDB 存储引擎的表才能保证数据一致性。
- 恢复数据时目标 MySQL 实例上 `local_infile` 系统变量的全局设置必须为 `ON`，若未开启，则无法完成恢复操作，如会出现下面现象：

```bash
 MySQL  localhost:33060+ ssl  JS > util.loadDump('/root/airport-db', { threads: 16, deferTableIndexes: 'all', ignoreVersion: true });
ERROR: The 'local_infile' global system variable must be set to ON in the target server, after the server is verified to be trusted.
Util.loadDump: local_infile disabled in server (MYSQLSH 53025)
 MySQL  localhost:33060+ ssl  JS >
```

参考：[官方文档](https://dev.mysql.com/doc/mysql-shell/8.0/en/mysql-shell-utilities-dump-instance-schema.html)

## （四）优缺点

**优点**

1. 比 mysqldump、mydumper 恢复速度快，这主要源于它的多线程支持
2. 功能丰富。

**缺点**

1. MySQL Shell 会经常更新，提供修复和新功能

    说白了就是不稳定，可能隐藏存在一些缺陷。 所以官方强烈建议始终使用最新版本。 最新版本的 MySQL Shell 可用于任何版本的 MySQL 5.7 或 8.0。

    [官方文档](https://dev.mysql.com/doc/mysql-shell/8.0/en/mysql-shell-features.html#mysql-shell-features-utilities)
    [官方Bug报告](https://bugs.mysql.com/search.php?search_for=mysql+shell+utilities&status=Active&severity=&limit=All&order_by=id&cmd=display&direction=DESC&os=0&phpver=&bug_age=0)

    比如：创建两个用户, 一个 dmpu1 用户使用数字字符串“123”作为密码，一个 dmpu2 用户会用复杂密码 “Hello@world#123”

    在使用数字作为密码连接时，提示“123” 不是字符串，但是计算实际行是字符串，只不过是数字形式的字符串。
    ```bash
    # ./mysqlsh -- util check-for-server-upgrade --user=dmpu1 --host=localhost --port=3306 --password='123' --outputFormat=JSON
    ERROR: Argument connectionData: Argument password is expected to be a string
    ```

    对比 dmpu2 用户使用的复杂密码，则无此现象：
    ```bash
    # ./mysqlsh -- util check-for-server-upgrade --user=dmpu1 --host=localhost --port=3306 --password='Hello@world#123' --outputFormat=JSON
    ```

2. 安装包较大 
    以 `mysql-shell-8.0.37-linux-glibc2.17-x86-64bit.tar.gz` 为例：

    压缩包：81.1M
    解压后：393M
    mysqlsh: 43M（MySQL 8.0.35 版本中 mysqldump 为 7.4M，mysqlpump 为 8.1M）

    若集成到 urman-agent 组件中，原本 urman-agent 不管是安装，还是升级，包传输过程原本就很慢，这无疑会带来更大的负担。

3. MySQL Shell 作为实用工具集，功能并不单一。

    它不单作为备份/恢复工具，还集成其他功能，比如支持支持JS、Python语言的交互式执行、API命令行的集成等功能。这也是导致它安装包较大的原因之一。
    如果是仅使用其中的某项功能，安装户时势必会携带其他的功能的安装。

4. 学习时间成本较高。

    由于工具较新，且更新较为频繁，需要我们时常关注官方的更新动态，并花费时间了解修复的问题和引入的新功能。

5. 集成成本（不考虑包大小）

    a)、进入 https://downloads.mysql.com/archives/shell/ 下载最新版本版本并上传至FTP指定目录。
    b)、urman 调整 Makefile，build.spec 构建包配置文件，在构建包时从ftp下载 mysql-shell 压缩包并解压到包urman-agent安装目录下，使rman-agent组件安装后直接可以进入bin目录下执行mysqlsh二进制程序。
    c)、集成成本约 0.5人天。


# 安装

1、下载

```bash
wget https://downloads.mysql.com/archives/get/p/43/file/mysql-shell-8.0.37-linux-glibc2.17-x86-64bit.tar.gz
```

2、解压安装

```bash
# tar zxvf mysql-shell-8.0.37-linux-glibc2.17-x86-64bit.tar.gz -C /opt/
# cd /opt/
# mv mysql-shell-8.0.37-linux-glibc2.17-x86-64bit mysql-shell && cd mysql-shell
# ./bin/mysqlsh --version
./bin/mysqlsh   Ver 8.0.37 for Linux on x86_64 - for MySQL 8.0.37 (MySQL Community Server (GPL))
```


mysqlsh -- util check-for-server-upgrade --user=root --host=localhost --port=3306 --password='123' --outputFormat=JSON --config-path=/etc/mysql/my.cnf

create user 'dmpu1'@'%' identified with mysql_native_password by 'Hello@world#123';
grant create, insert, select, reload, process, lock tables, replication client, replication_slave_admin, show view, trigger, backup_admin, super on *.* to dmpu1@'%';


create user 'dmpu2'@'%' identified with mysql_native_password by '123';
grant create, insert, select, reload, process, lock tables, replication client, replication_slave_admin, show view, trigger, backup_admin, super on *.* to dmpu2@'%';


./mysqlsh -- util check-for-server-upgrade --user=dmpu1 --host=localhost --port=3306 --password='Hello@world#123' --outputFormat=JSON
./mysqlsh -- util check-for-server-upgrade --user=dmpu2 --host=localhost --port=3306 --password='123' --outputFormat=JSON

# 使用

这里使用的不是交互式环境，而是使用的API命令行集成环境，参考：https://dev.mysql.com/doc/mysql-shell/8.0/en/command-line-integration-overview.html

语法：
```bash
mysqlsh [options] -- [shell_object]+ object_method [arguments]
```

查看帮忙：
```bash
mysqlsh --help
```

查看帮助：
```bash
# ./mysqlsh  -- --help
The following objects provide command line operations:

   cluster
      Represents an InnoDB Cluster.

   clusterset
      Represents an InnoDB ClusterSet.

   dba
      InnoDB Cluster, ReplicaSet, and ClusterSet management functions.

   rs
      Represents an InnoDB ReplicaSet.

   shell
      Gives access to general purpose functions and properties.

   util
      Global object that groups miscellaneous tools like upgrade checker and
      JSON import.
#
```

再比如：
```bash
# ./mysqlsh  -- util --help
The following object provides command line operations at 'util':

   debug
      Debugging and diagnostic utilities.

The following operations are available at 'util':

   check-for-server-upgrade
      Performs series of tests on specified MySQL server to check if the
      upgrade process will succeed.

   dump-instance
      Dumps the whole database to files in the output directory.

   dump-schemas
      Dumps the specified schemas to the files in the output directory.

   dump-tables
      Dumps the specified tables or views from the given schema to the files in
      the target directory.

   export-table
      Exports the specified table to the data dump file.

   import-json
      Import JSON documents from file to collection or table in MySQL Server
      using X Protocol session.

   import-table
      Import table dump stored in files to target table using LOAD DATA LOCAL
      INFILE calls in parallel connections.

   load-dump
      Loads database dumps created by MySQL Shell.
#
# ./mysqlsh  -- util dump-schemas --help
```

## 数据准备

1、开启恢复实例 `local_infile`：

```sql
mysql> show global variables like '%local%';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| local_infile  | OFF   |
+---------------+-------+
1 row in set (0.00 sec)

mysql> set global local_infile=on;
Query OK, 0 rows affected (0.00 sec)

mysql> show global variables like '%local%';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| local_infile  | ON    |
+---------------+-------+
1 row in set (0.00 sec)

mysql>
```

2、导入

我导入的使用MySQL官网提供的 employee data 和 sakila database 两个库。

```
./mysqlsh  -- util dump-schemas --help


util dump-schemas <schemas> --outputUrl=<str> [<options>]

./mysqlsh  -- util dump-schemas ['backup1'] --user=root --host=localhost --port=3306 --password='123' --outputUrl='/data/backup/mysql-shell/backup1_schema'

./mysqlsh --user=root --port=3306 --password='123' -- util dump-schemas ['backup1'] --outputUrl='/data/backup/mysql-shell/backup1_schema'

./mysqlsh --user=root --port=3306 --password='123' -- util dump-schemas 'backup1' --outputUrl='/data/backup/mysql-shell/backup1_schema'


util.dumpSchemas(['backup1'],'/data/backup/mysql-shell/backup1_schema')
```

## 常用方法

```
util.dumpInstance(outputUrl[, options]) 
util.dumpSchemas(schemas, outputUrl[, options])
util.dumpTables(schema, tables, outputUrl[, options])
```

### dump-instance

    Dumps the whole database to files in the output directory.

**语法**

```bash
util dump-instance <outputUrl> [<options>]
```

**示例**

```bash
./mysqlsh --socket=/opt/mysql/data/3306/mysqld.sock -uroot -P3306 -p'123' -- util dump-instance /opt/mysql/backup/instance01
```

完成过程如下：
```bash
# ./mysqlsh --socket=/opt/mysql/data/3306/mysqld.sock -uroot -P3306 -p'123' -- util dump-instance /opt/mysql/backup/instance01
WARNING: Using a password on the command line interface can be insecure.
NOTE: Backup lock is not supported in MySQL 5.7 and DDL changes will not be blocked. The dump may fail with an error if schema changes are made while dumping.
Acquiring global read lock
Global read lock acquired
Initializing - done
3 out of 7 schemas will be dumped and within them 23 tables, 9 views, 6 routines, 6 triggers.
3 out of 5 users will be dumped.
Gathering information - done
All transactions have been started
Global read lock has been released
Writing global DDL files
Writing users DDL
WARNING: User 'universe_op'@'%' has a grant statement on an object which is not included in the dump (GRANT EXECUTE ON `sys`.* TO 'universe_op'@'%' WITH GRANT OPTION)
WARNING: User 'universe_op'@'%' has a grant statement on an object which is not included in the dump (GRANT SELECT, UPDATE ON `mysql`.* TO 'universe_op'@'%' WITH GRANT OPTION)
Running data dump using 4 threads.
NOTE: Progress information uses estimated values and may not be accurate.
Writing schema metadata - done
Writing DDL - done
Writing table metadata - done
Starting data dump
100% (3.97M rows / ~3.96M rows), 5.89M rows/s, 0.00 B/s uncompressed, 0.00 B/s compressed
Dump duration: 00:00:00s
Total duration: 00:00:00s
Schemas dumped: 3
Tables dumped: 23
Uncompressed data size: 144.53 MB
Compressed data size: 37.70 MB
Compression ratio: 3.8
Rows written: 3966283
Bytes written: 37.70 MB
Average uncompressed throughput: 144.53 MB/s
Average compressed throughput: 37.70 MB/s
```

如果需要压缩：
```bash
./mysqlsh --socket=/opt/mysql/data/3306/mysqld.sock -uroot -P3306 -p'123' -- util dump-instance /opt/mysql/backup/instance02 --compression=gzip
```

**注意**
1、如果备份实例目录不存在，它会自动帮忙创建目录，例如上面的 “instance01” 目录。
2、备份目录必须为空，否则报错。报错提示形式如下：
“ERROR: Cannot proceed with the dump, the specified directory '/opt/mysql/backup/instance01' already exists at the target location /opt/mysql/backup/instance01 and is not empty.”


1. urman-mgr 提供任务同步服务
任务的编排来源于备份规则
2、


- outputUrl: 备份目录

先创建备份用户：
```sql
create user 'u_utility'@'%' identified with mysql_native_password by '123';
```

授权：
```sql
grant create, insert, select, reload, process, lock tables, replication client, replication_slave_admin, show view, trigger, event, backup_admin, super on *.* to u_utility@'%';
```



备份:

```bash
 MySQL  127.0.0.1:3306 ssl  JS > util.dumpInstance('/data/backup/mysql-shell/full')
Acquiring global read lock
Global read lock acquired
Initializing - done
2 out of 6 schemas will be dumped and within them 3 tables, 0 views.
5 out of 8 users will be dumped.
Gathering information - done
All transactions have been started
Locking instance for backup
Global read lock has been released
Writing global DDL files
Writing users DDL
Running data dump using 4 threads.
NOTE: Progress information uses estimated values and may not be accurate.
Writing schema metadata - done
Writing DDL - done
Writing table metadata - done
Starting data dump
100% (6 rows / ~6 rows), 0.00 rows/s, 0.00 B/s uncompressed, 0.00 B/s compressed
Dump duration: 00:00:00s
Total duration: 00:00:00s
Schemas dumped: 2
Tables dumped: 3
Uncompressed data size: 60 bytes
Compressed data size: 87 bytes
Compression ratio: 0.7
Rows written: 6
Bytes written: 87 bytes
Average uncompressed throughput: 60.00 B/s
Average compressed throughput: 87.00 B/s
 MySQL  127.0.0.1:3306 ssl  JS >

查看备份内容：
```bash
[root@chaos-1 mysql-shell]# ll full/
total 88
-rw-r----- 1 root root  317 9月   9 00:22 backup1.json
-rw-r----- 1 root root  569 9月   9 00:22 backup1.sql
-rw-r----- 1 root root   29 9月   9 00:22 backup1@t1@@0.tsv.zst
-rw-r----- 1 root root    8 9月   9 00:22 backup1@t1@@0.tsv.zst.idx
-rw-r----- 1 root root  629 9月   9 00:22 backup1@t1.json
-rw-r----- 1 root root  739 9月   9 00:22 backup1@t1.sql
-rw-r----- 1 root root   29 9月   9 00:22 backup1@t2@@0.tsv.zst
-rw-r----- 1 root root    8 9月   9 00:22 backup1@t2@@0.tsv.zst.idx
-rw-r----- 1 root root  629 9月   9 00:22 backup1@t2.json
-rw-r----- 1 root root  739 9月   9 00:22 backup1@t2.sql
-rw-r----- 1 root root  275 9月   9 00:22 backup2.json
-rw-r----- 1 root root  569 9月   9 00:22 backup2.sql
-rw-r----- 1 root root   29 9月   9 00:22 backup2@t1@@0.tsv.zst
-rw-r----- 1 root root    8 9月   9 00:22 backup2@t1@@0.tsv.zst.idx
-rw-r----- 1 root root  629 9月   9 00:22 backup2@t1.json
-rw-r----- 1 root root  739 9月   9 00:22 backup2@t1.sql
-rw-r----- 1 root root  356 9月   9 00:22 @.done.json
-rw-r----- 1 root root  984 9月   9 00:22 @.json
-rw-r----- 1 root root  240 9月   9 00:22 @.post.sql
-rw-r----- 1 root root  240 9月   9 00:22 @.sql
-rw-r----- 1 root root 4391 9月   9 00:22 @.users.sql
[root@chaos-1 mysql-shell]#
```

查看数据

`backup1.sql` 建库信息
`backup1@t1@@0.tsv.zst` 数据信息


**备份指定schema**


/bin/mysqlsh --scoket=/opt/mysql/data/3306/mysqld.sock -uroot -p3306 -p123 -- util dump-schemas 'sakila' --outputUrl='/data/backup/mysql-shell/backup1_schema'
/opt/mysql-shell/bin/mysqlsh --user=root --port=3306 --password='123' -- util dump-schemas 'backup1' --outputUrl='/data/backup/mysql-shell/backup1_schema'

```bash
[root@chaos-1 backup]# /opt/mysql-shell/bin/mysqlsh --user=root --port=3306 --password='123' -- util dump-schemas 'backup1' --outputUrl='/data/backup/mysql-shell/backup1_schema'
WARNING: Using a password on the command line interface can be insecure.
Acquiring global read lock
Global read lock acquired
Initializing - done
1 schemas will be dumped and within them 1 table, 0 views.
Gathering information - done
All transactions have been started
Locking instance for backup
Global read lock has been released
Writing global DDL files
Running data dump using 4 threads.
NOTE: Progress information uses estimated values and may not be accurate.
Writing schema metadata - done
Writing DDL - done
Writing table metadata - done
Starting data dump
100% (2 rows / ~2 rows), 0.00 rows/s, 0.00 B/s uncompressed, 0.00 B/s compressed
Dump duration: 00:00:00s
Total duration: 00:00:00s
Schemas dumped: 1
Tables dumped: 1
Uncompressed data size: 20 bytes
Compressed data size: 29 bytes
Compression ratio: 0.7
Rows written: 2
Bytes written: 29 bytes
Average uncompressed throughput: 20.00 B/s
Average compressed throughput: 29.00 B/s
[root@chaos-1 backup]#
```

删除 backup1 


恢复：
```bash
[root@chaos-1 backup]# /opt/mysql-shell/bin/mysqlsh --user=root --port=3306 --password='123' -- util load-dump '/data/backup/mysql-shell/backup1_schema'
WARNING: Using a password on the command line interface can be insecure.
Loading DDL and Data from '/data/backup/mysql-shell/backup1_schema' using 4 threads.
Opening dump...
Target is MySQL 8.0.39. Dump was produced from MySQL 8.0.39
Scanning metadata - done
Checking for pre-existing objects...
Executing common preamble SQL
Executing DDL - done
Executing view DDL - done
Starting data load
Executing common postamble SQL
100% (20 bytes / 20 bytes), 0.00 B/s, 1 / 1 tables done
Recreating indexes - done
1 chunks (2 rows, 20 bytes) for 1 tables in 1 schemas were loaded in 0 sec (avg throughput 20.00 B/s)
0 warnings were reported during the load.
[root@chaos-1 backup]#
```


**备份指定表**
```bash
util dump-tables <schema> <tables> --outputUrl=<str> [<options>]
[root@chaos-1 backup]# /opt/mysql-shell/bin/mysqlsh --user=root --port=3306 --password='123' -- util dump-tables 'backup1' 't2'  --outputUrl='/data/backup/mysql-shell/backup1_t2'
WARNING: Using a password on the command line interface can be insecure.
Acquiring global read lock
Global read lock acquired
Initializing - done
1 tables and 0 views will be dumped.
Gathering information - done
All transactions have been started
Locking instance for backup
Global read lock has been released
Writing global DDL files
Running data dump using 4 threads.
NOTE: Progress information uses estimated values and may not be accurate.
Writing schema metadata - done
Writing DDL - done
Writing table metadata - done
Starting data dump
?% (0 rows / ?), 0.00 rows/s, 0.00 B/s uncompressed, 0.00 B/s compressed
Dump duration: 00:00:00s
Total duration: 00:00:00s
Schemas dumped: 1
Tables dumped: 1
Uncompressed data size: 0 bytes
Compressed data size: 9 bytes
Compression ratio: 0.0
Rows written: 0
Bytes written: 9 bytes
Average uncompressed throughput: 0.00 B/s
Average compressed throughput: 9.00 B/s
[root@chaos-1 backup]#
```


恢复：
```
[root@chaos-1 backup]#  /opt/mysql-shell/bin/mysqlsh --user=root --port=3306 --password='123' -- util load-dump '/data/backup/mysql-shell/backup1_t2' --includeTables='backup1.t2'
WARNING: Using a password on the command line interface can be insecure.
Loading DDL and Data from '/data/backup/mysql-shell/backup1_t2' using 4 threads.
Opening dump...
Target is MySQL 8.0.39. Dump was produced from MySQL 8.0.39
Scanning metadata - done
Checking for pre-existing objects...
Executing common preamble SQL
Executing DDL - done
Executing view DDL - done
Starting data load
?% (0 bytes / ?), 0.00 B/s, 0 / 1 tables done
Recreating indexes - done
Executing common postamble SQL
No data loaded.
0 warnings were reported during the load.
```




**备份指定库**

```bash
 MySQL  127.0.0.1:3306 ssl  JS > util.dumpSchemas(['backup1'],'/data/backup/mysql-shell/backup1_schema')
Acquiring global read lock
Global read lock acquired
Initializing - done
1 schemas will be dumped and within them 2 tables, 0 views.
Gathering information - done
All transactions have been started
Locking instance for backup
Global read lock has been released
Writing global DDL files
Running data dump using 4 threads.
NOTE: Progress information uses estimated values and may not be accurate.
Writing schema metadata - done
Writing DDL - done
Writing table metadata - done
Starting data dump
100% (4 rows / ~4 rows), 0.00 rows/s, 0.00 B/s uncompressed, 0.00 B/s compressed
Dump duration: 00:00:00s
Total duration: 00:00:00s
Schemas dumped: 1
Tables dumped: 2
Uncompressed data size: 40 bytes
Compressed data size: 58 bytes
Compression ratio: 0.7
Rows written: 4
Bytes written: 58 bytes
Average uncompressed throughput: 40.00 B/s
Average compressed throughput: 58.00 B/s
 MySQL  127.0.0.1:3306 ssl  JS >
```

查看备份内容：
```bash
[root@chaos-1 mysql-shell]# ll backup1_schema
total 56
-rw-r----- 1 root root 317 9月   9 00:28 backup1.json
-rw-r----- 1 root root 569 9月   9 00:28 backup1.sql
-rw-r----- 1 root root  29 9月   9 00:28 backup1@t1@@0.tsv.zst
-rw-r----- 1 root root   8 9月   9 00:28 backup1@t1@@0.tsv.zst.idx
-rw-r----- 1 root root 629 9月   9 00:28 backup1@t1.json
-rw-r----- 1 root root 739 9月   9 00:28 backup1@t1.sql
-rw-r----- 1 root root  29 9月   9 00:28 backup1@t2@@0.tsv.zst
-rw-r----- 1 root root   8 9月   9 00:28 backup1@t2@@0.tsv.zst.idx
-rw-r----- 1 root root 629 9月   9 00:28 backup1@t2.json
-rw-r----- 1 root root 739 9月   9 00:28 backup1@t2.sql
-rw-r----- 1 root root 266 9月   9 00:28 @.done.json
-rw-r----- 1 root root 770 9月   9 00:28 @.json
-rw-r----- 1 root root 240 9月   9 00:28 @.post.sql
-rw-r----- 1 root root 240 9月   9 00:28 @.sql
[root@chaos-1 mysql-shell]#
```

**备份指定表**

```bash
 MySQL  127.0.0.1:3306 ssl  JS > util.dumpTables('backup1',['t2'],'/data/backup/mysql-shell/backup1_t2')
Acquiring global read lock
Global read lock acquired
Initializing - done
1 tables and 0 views will be dumped.
Gathering information - done
All transactions have been started
Locking instance for backup
Global read lock has been released
Writing global DDL files
Running data dump using 4 threads.
NOTE: Progress information uses estimated values and may not be accurate.
Writing schema metadata - done
Writing DDL - done
Writing table metadata - done
Starting data dump
100% (2 rows / ~2 rows), 0.00 rows/s, 0.00 B/s uncompressed, 0.00 B/s compressed
Dump duration: 00:00:00s
Total duration: 00:00:00s
Schemas dumped: 1
Tables dumped: 1
Uncompressed data size: 20 bytes
Compressed data size: 29 bytes
Compression ratio: 0.7
Rows written: 2
Bytes written: 29 bytes
Average uncompressed throughput: 20.00 B/s
Average compressed throughput: 29.00 B/s
 MySQL  127.0.0.1:3306 ssl  JS >
```

查看备份内容：
```bash
[root@chaos-1 mysql-shell]# ll backup1_t2
total 40
-rw-r----- 1 root root 214 9月   9 00:32 backup1.json
-rw-r----- 1 root root 470 9月   9 00:32 backup1.sql
-rw-r----- 1 root root  29 9月   9 00:32 backup1@t2@@0.tsv.zst
-rw-r----- 1 root root   8 9月   9 00:32 backup1@t2@@0.tsv.zst.idx
-rw-r----- 1 root root 629 9月   9 00:32 backup1@t2.json
-rw-r----- 1 root root 739 9月   9 00:32 backup1@t2.sql
-rw-r----- 1 root root 207 9月   9 00:32 @.done.json
-rw-r----- 1 root root 769 9月   9 00:32 @.json
-rw-r----- 1 root root 240 9月   9 00:32 @.post.sql
-rw-r----- 1 root root 240 9月   9 00:32 @.sql
[root@chaos-1 mysql-shell]#
```

**展示备份进度**

```bash
 MySQL  127.0.0.1:3306 ssl  JS > util.dumpSchemas(['backup1'],'/data/backup/mysql-shell/backup1_schema_progress', {showProgress: true})
Acquiring global read lock
Global read lock acquired
Initializing - done
1 schemas will be dumped and within them 2 tables, 0 views.
Gathering information - done
All transactions have been started
Locking instance for backup
Global read lock has been released
Writing global DDL files
Running data dump using 4 threads.
NOTE: Progress information uses estimated values and may not be accurate.
Writing schema metadata - done
Writing DDL - done
Writing table metadata - done
Starting data dump
100% (4 rows / ~4 rows), 0.00 rows/s, 0.00 B/s uncompressed, 0.00 B/s compressed
Dump duration: 00:00:00s
Total duration: 00:00:00s
Schemas dumped: 1
Tables dumped: 2
Uncompressed data size: 40 bytes
Compressed data size: 58 bytes
Compression ratio: 0.7
Rows written: 4
Bytes written: 58 bytes
Average uncompressed throughput: 40.00 B/s
Average compressed throughput: 58.00 B/s
 MySQL  127.0.0.1:3306 ssl  JS >
```

**恢复**



恢复：
```bash
SET GLOBAL local_infile = ON;
util.loadDump("/data/backup/mysql-shell/full", {showProgress:true})
```



# ./bin/mysqlsh -S /tmp/mysql.sock 
mysql-js> util.loadDump("/data/backup/full",{loadUsers: true})
util.loadDump("/data/backup/universe",{loadUsers: true})




[root@-udp-1 bin]# ./mysqlsh -S /opt/mysql/data/3306/mysqld.sock -uroot -P3306 -p123
MySQL Shell 8.0.38

Copyright (c) 2016, 2024, Oracle and/or its affiliates.
Oracle is a registered trademark of Oracle Corporation and/or its affiliates.
Other names may be trademarks of their respective owners.

Type '\help' or '\?' for help; '\quit' to exit.
WARNING: Using a password on the command line interface can be insecure.
Creating a session to 'root@localhost:3306'
Fetching schema names for auto-completion... Press ^C to stop.
Your MySQL connection id is 404
Server version: 5.7.25-log MySQL Community Server (GPL)
No default schema selected; type \use <schema> to set one.
 MySQL  localhost:3306  JS >

 MySQL  localhost:3306  JS > util.loadDump("/data/backup/universe",{loadUsers: true})
Loading DDL, Data and Users from '/data/backup/universe' using 4 threads.
Opening dump...
WARNING: The 'loadUsers' option is set to true, but the dump does not contain the user data.
Target is MySQL 5.7.25-log. Dump was produced from MySQL 5.7.25-log
Scanning metadata - done
Checking for pre-existing objects...
Executing common preamble SQL
Executing DDL - done
Executing user accounts SQL...
Executing view DDL - done
Starting data load
Executing common postamble SQL
100% (130.08 KB / 130.08 KB), 0.00 B/s, 17 / 17 tables done
Recreating indexes - done
17 chunks (39 rows, 130.08 KB) for 17 tables in 1 schemas were loaded in 0 sec (avg throughput 130.08 KB/s)
1 accounts were loaded
0 warnings were reported during the load.
 MySQL  localhost:3306  JS >
```


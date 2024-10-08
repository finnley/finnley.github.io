+++
title = 'Ubuntu16.04搭建NodeJs环境Ubuntu中通过Docker安装配置MySQL主从节点'
date = 2021-06-14T21:21:24+08:00
draft = true
categories = [ "Ubuntu" ]
tags = [ "ubuntu" ]
+++

    (以下docker相关的命令，需要在root用户环境下或通过sudo提升权限来进行操作。)

## 1. 拉取MySQL5.7镜像到本地

```
docker pull mysql:5.7
```

* 如果你只需要跑一个mysql实例，不做主从，那么执行以下命令即可，不用再做后面的参考步骤

```
docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql:5.7
```

*  然后用shell或客户端软件通过配置( 用户名:root 密码:132456 IP:你的本机ip 端口:3306)来登录即可

## 2. 准备MySQL配置文件

`mysql5.7` 安装后的默认配置文件在 `/etc/mysql/my.cnf`, 而自定义的配置文件一般放在 `/etc/mysql/conf.d` 这个路径下。
现在我们在本地host主机上自定义的某个目录(如 `/data/mysql/conf/` )，先创建两个文件master.conf和slave.conf，分别用于配置主从两个节点。

* /data/mysql/conf/master.conf

```
[client]
default-character-set=utf8
[mysql]
default-character-set=utf8
[mysqld]
log_bin = log  #开启二进制日志，用于从节点的历史复制回放
collation-server = utf8_unicode_ci
init-connect='SET NAMES utf8'
character-set-server = utf8
server_id = 1  #需保证主库和从库的server_id不同， 假设主库设为1
replicate-do-db=fileserver  #需要复制的数据库名，需复制多个数据库的话则重复设置这个选项 (如果想同步所有的数据库，则直接注释这一行配置)
```

* /data/mysql/conf/slave.conf

```
[client]
default-character-set=utf8
[mysql]
default-character-set=utf8
[mysqld]
log_bin = log  #开启二进制日志，用于从节点的历史复制回放
collation-server = utf8_unicode_ci
init-connect='SET NAMES utf8'
character-set-server = utf8
server_id = 2  #需保证主库和从库的server_id不同， 假设从库设为2
replicate-do-db=fileserver  #需要复制的数据库名，需复制多个数据库的话则重复设置这个选项 (如果想同步所有的数据库，则直接注释这一行配置)
```

## 3. Docker分别运行MySQL主/从两个容器

* 将mysql主节点运行起来

```
mkdir -p /data/mysql/datam
docker run -d --name mysql-master -p 13306:3306 -v /data/mysql/conf/master.conf:/etc/mysql/mysql.conf.d/mysqld.cnf -v /data/mysql/datam:/var/lib/mysql  -e MYSQL_ROOT_PASSWORD=123456 mysql:5.7
```

运行参数说明:

```
–name mysql-master: 容器的名称设为mysql-master
-p 13306:3306: 将host的13306端口映射到容器的3306端口
-v /data/mysql/conf/master.conf:/etc/mysql/mysql.conf.d/mysqld.cnf ： master.conf配置文件挂载
-v /data/mysql/datam:/var/lib/mysql ： mysql容器内数据挂载到host的/data/mysql/datam， 用于持久化
-e MYSQL_ROOT_PASSWORD=123456 : mysql的root登录密码为123456
```

* 将mysql从节点运行起来

```
mkdir -p /data/mysql/datas
docker run -d --name mysql-slave -p 13307:3306 -v /data/mysql/conf/slave.conf:/etc/mysql/mysql.conf.d/mysqld.cnf -v /data/mysql/datas:/var/lib/mysql  -e MYSQL_ROOT_PASSWORD=123456 mysql:5.7
```

运行参数说明:

```
–name mysql-slave: 容器的名称设为mysql-slave
-p 13307:3306: 将host的13307端口映射到容器的3306端口
-v /data/mysql/conf/slave.conf:/etc/mysql/mysql.conf.d/mysqld.cnf ： slave.conf配置文件挂载
-v /data/mysql/datas:/var/lib/mysql ： mysql容器内数据挂载到host的/data/mysql/datas， 用于持久化
-e MYSQL_ROOT_PASSWORD=123456 : mysql的root登录密码为123456
```

## 4. 登录MySQL主节点配置同步信息

* 宿主机安装mysql客户端

```
sudo apt-get install -y mysql-client
```

* 登录mysql

- 192.168.1.xx 是你本机的内网ip

```
mysql -u root -h 192.168.1.xx -P13306 -p123456
```

* 在mysql client中执行 (创建用于访问主节点来同步数据的帐号)

```
mysql> create user slave identified by 'slave';
mysql> GRANT REPLICATION SLAVE ON *.* TO 'slave'@'%' IDENTIFIED BY 'slave';
mysql> flush privileges;
mysql> create database fileserver default character set utf8mb4;
```

* 再获取status, 得到类似如下的输出:

```
mysql> show master status \G;
*************************** 1. row ***************************
             File: log.000003
         Position: 1035
     Binlog_Do_DB:
 Binlog_Ignore_DB:
Executed_Gtid_Set:
1 row in set (0.00 sec)
```

## 5. 登录MySQL从节点配置同步信息

* 查看mysql master的容器独立ip地址，比如输出得到: 172.17.0.2

```
docker inspect --format='{{.NetworkSettings.IPAddress}}' mysql-master
```

* 另开一个tab登录mysql，192.168.1.xx 是你本机的内网ip

```
mysql -u root -h 192.168.1.xx -P13307 -p123456
```

* 在mysql client中操作:

```
mysql> stop slave;
mysql> create database fileserver default character set utf8mb4;
#注意其中的日志文件和数值要和上面show master status的值对应
mysql> CHANGE MASTER TO MASTER_HOST='前两个步骤中获得的mysql master ip',MASTER_PORT=3306,MASTER_USER='slave',MASTER_PASSWORD='slave',MASTER_LOG_FILE='log.000025',MASTER_LOG_POS=155;
mysql> start slave;
```

* 再获取status, 正常应该得到类似如下的输出:

```
mysql> show slave status \G;
*************************** 1. row ***************************
               Slave_IO_State:
                  Master_Host: 172.17.0.2
                  Master_User: slave
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: log.000025
          Read_Master_Log_Pos: 155
               Relay_Log_File: 8aeef24c3aa1-relay-bin.000001
                Relay_Log_Pos: 4
        Relay_Master_Log_File: log.000025
             Slave_IO_Running: No
            Slave_SQL_Running: Yes
              Replicate_Do_DB: fileserver
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 155
              Relay_Log_Space: 154
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File:
           Master_SSL_CA_Path:
              Master_SSL_Cert:
            Master_SSL_Cipher:
               Master_SSL_Key:
        Seconds_Behind_Master: NULL
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 1236
                Last_IO_Error: Got fatal error 1236 from master when reading data from binary log: 'Could not find first log file name in binary log index file'
               Last_SQL_Errno: 0
               Last_SQL_Error:
  Replicate_Ignore_Server_Ids:
             Master_Server_Id: 1
                  Master_UUID: bf5d8626-cd15-11eb-842a-0242ac110002
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind:
      Last_IO_Error_Timestamp: 210614 13:59:34
     Last_SQL_Error_Timestamp:
               Master_SSL_Crl:
           Master_SSL_Crlpath:
           Retrieved_Gtid_Set:
            Executed_Gtid_Set:
                Auto_Position: 0
         Replicate_Rewrite_DB:
                 Channel_Name:
           Master_TLS_Version:
1 row in set (0.00 sec)
```

到这时说明主从配置已经完成，可以尝试在主mysql的fileserver数据库里建表操作下，然后在从节点上检查数据是否已经同步过来。

我们也可以通过Docker Compose的方式来安装MySQL主从集群:

https://github.com/mayinghan/mysql-master-slave-service

## MySQL 主从数据同步演示

* 登录master

```
mysql -u root -h 192.168.1.xx -P13306 -p123456
```

```
show master status;
+------------+----------+--------------+------------------+-------------------+
| File       | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------+----------+--------------+------------------+-------------------+
| log.000003 |     1035 |              |                  |                   |
+------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```

* 另开一个终端登录slave

```
mysql -u root -h 192.168.1.xx -P13307 -p123456
```

```
mysql> CHANGE MASTER TO MASTER_HOST='192.168.33.33',MASTER_USER='reader',MASTER_PASSWORD='reader',MASTER_LOG_FILE='binlog.000003',MASTER_LOG_POS=0;
Query OK, 0 rows affected, 1 warning (0.01 sec)

mysql> start slave;
Query OK, 0 rows affected (0.00 sec)

mysql> show slave status \G;
*************************** 1. row ***************************
               Slave_IO_State: Connecting to master
                  Master_Host: 192.168.33.33
                  Master_User: reader
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: binlog.000003
          Read_Master_Log_Pos: 4
               Relay_Log_File: 4079e373a619-relay-bin.000001
                Relay_Log_Pos: 4
        Relay_Master_Log_File: binlog.000003
             Slave_IO_Running: Connecting
            Slave_SQL_Running: Yes
              Replicate_Do_DB: fileserver
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 4
              Relay_Log_Space: 154
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File:
           Master_SSL_CA_Path:
              Master_SSL_Cert:
            Master_SSL_Cipher:
               Master_SSL_Key:
        Seconds_Behind_Master: NULL
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 2003
                Last_IO_Error: error connecting to master 'reader@192.168.33.33:3306' - retry-time: 60  retries: 1
               Last_SQL_Errno: 0
               Last_SQL_Error:
  Replicate_Ignore_Server_Ids:
             Master_Server_Id: 0
                  Master_UUID:
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind:
      Last_IO_Error_Timestamp: 210614 23:27:29
     Last_SQL_Error_Timestamp:
               Master_SSL_Crl:
           Master_SSL_Crlpath:
           Retrieved_Gtid_Set:
            Executed_Gtid_Set:
                Auto_Position: 0
         Replicate_Rewrite_DB:
                 Channel_Name:
           Master_TLS_Version:
1 row in set (0.00 sec)
```
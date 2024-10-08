+++
title = 'MySQL安装'
date = 2024-05-03T14:57:45+08:00
draft = true
categories = [ "MySQL" ]
tags = [ "mysql" ]
+++

## 文档

[Installing MySQL on Unix/Linux Using Generic Binaries](https://dev.mysql.com/doc/refman/5.7/en/binary-installation.html)
[Postinstallation Setup and Testing](https://dev.mysql.com/doc/refman/5.7/en/postinstallation.html)
[Initializing the Data Directory](https://dev.mysql.com/doc/refman/5.7/en/data-directory-initialization.html)

## 下载MySQL

[下载地址](https://downloads.mysql.com/archives/community/)

```shell
wget https://downloads.mysql.com/archives/get/p/23/file/mysql-5.7.41-linux-glibc2.12-x86_64.tar.gz
sudo mv mysql-5.7.41-linux-glibc2.12-x86_64.tar.gz /usr/local
```

## 安装依赖

```
yum install -y libaio
```

## 添加mysql组及用户

```
# groupadd mysql
# useradd -r -g mysql -s /bin/false mysql
```

## 解压

```shell
# cd /usr/local
# tar zxvf mysql-5.7.41-linux-glibc2.12-x86_64.tar.gz
# ls -al mysql-5.7.41-linux-glibc2.12-x86_64
total 268
drwxr-xr-x.  9 root root     129 Apr 30 08:40 .
drwxr-xr-x. 13 root root     224 Apr 30 08:40 ..
drwxr-xr-x.  2 root root    4096 Apr 30 08:40 bin
drwxr-xr-x.  2 root root      73 Apr 30 08:40 docs
drwxr-xr-x.  3 root root    4096 Apr 30 08:40 include
drwxr-xr-x.  5 root root     230 Apr 30 08:40 lib
-rw-r--r--.  1 7161 31415 255730 Dec  8 02:10 LICENSE
drwxr-xr-x.  4 root root      30 Apr 30 08:40 man
-rw-r--r--.  1 7161 31415    566 Dec  8 02:10 README
drwxr-xr-x. 28 root root    4096 Apr 30 08:40 share
drwxr-xr-x.  2 root root      90 Apr 30 08:40 support-files
```

## 建立软链

```shell
# ln -s mysql-5.7.41-linux-glibc2.12-x86_64 mysql
```

## 建立文件目录并设置权限

```shell
# cd mysql
# mkdir mysql-files
# chown mysql:mysql mysql-files
# chmod 750 mysql-files
```

## 初始化

使用mysql用户初始化数据库，初始化过程会为 root@localhost 生成一个临时密码，最后一行就是用来登录的临时密码：

```shell
# bin/mysqld --initialize --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data
2023-05-02T01:06:13.403769Z 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2023-05-02T01:06:13.647939Z 0 [Warning] InnoDB: New log files created, LSN=45790
2023-05-02T01:06:13.687127Z 0 [Warning] InnoDB: Creating foreign key constraint system tables.
2023-05-02T01:06:13.750536Z 0 [Warning] No existing UUID has been found, so we assume that this is the first time that this server has been started. Generating a new UUID: 888bb58a-e885-11ed-8835-5254004d77d3.
2023-05-02T01:06:13.751703Z 0 [Warning] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
2023-05-02T01:06:13.934215Z 0 [Warning] A deprecated TLS version TLSv1 is enabled. Please use TLSv1.2 or higher.
2023-05-02T01:06:13.934252Z 0 [Warning] A deprecated TLS version TLSv1.1 is enabled. Please use TLSv1.2 or higher.
2023-05-02T01:06:13.935139Z 0 [Warning] CA certificate ca.pem is self signed.
2023-05-02T01:06:14.007805Z 1 [Note] A temporary password is generated for root@localhost: 4Es7.tFNzsr!
```

接着初始化秘钥，在mysql通信过程中需要私钥和公钥进行加密，可以通过 `bin/mysql_ssl_rsa_setup` 生成初始秘钥:

```shell
# bin/mysql_ssl_rsa_setup --datadir=/usr/local/mysql/data
```

## 安装检查是否已存在安装包

```shell
# ll /etc/my.cnf
-rw-r--r--. 1 root root 570 Nov 28  2019 /etc/my.cnf
# rpm -qa | grep mariadb
mariadb-libs-5.5.65-1.el7.x86_64
# cp /etc/my.cnf /etc/my.cnf.bak
```

发现已经有了个mariadb-libs将其删除，删除的时候指定的 postfix 是mariadb-libs的一个依赖包，所以一并删除：

```shell
# rpm -e mariadb-libs postfix
$ ll /etc/my.cnf
ls: cannot access /etc/my.cnf: No such file or directory
```

如果不删除，在重新安装MySQL后起作用的将是 mariadb 的配置，初始化的时候就是以 mariadb 的配置进行初始化的
```shell
$ cat /etc/my.cnf
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
# Settings user and group are ignored when systemd is used.
# If you need to run mysqld under a different user or group,
# customize your systemd unit file for mariadb according to the
# instructions in http://fedoraproject.org/wiki/Systemd

[mysqld_safe]
log-error=/var/log/mariadb/mariadb.log
pid-file=/var/run/mariadb/mariadb.pid

#
# include all files from the config directory
#
!includedir /etc/my.cnf.d

```

临时配置：

```shell
[client]
port = 3306
socket = /usr/local/mysql/data/mysql.sock

[mysqld]
basedir = /usr/local/mysql
datadir = /usr/local/mysql/data
socket = /usr/local/mysql/data/mysql.sock
# symbolic-links=0

[mysqld_safe]
log-error=
pid-file=
```


## 启动MySQL

方法一：

```shell
$ sudo bin/mysqld_safe --user=mysql &
[1] 28274
[vagrant@centos-mysql mysql]$ Logging to '/usr/local/mysql-5.7.41-linux-glibc2.12-x86_64/data/centos-mysql.err'.
2023-05-02T01:11:39.030085Z mysqld_safe Starting mysqld daemon with databases from /usr/local/mysql-5.7.41-linux-glibc2.12-x86_64/data
```

从给出的信息可以知道启动 MySQL 的数据目录是存放在 /usr/local/mysql-5.7.41-linux-glibc2.12-x86_64/data 下

启动后查看下进程：

```shell
$ ps -ef | grep mysqld
root     28274 28073  0 09:11 pts/0    00:00:00 sudo bin/mysqld_safe --user=mysql
root     28276 28274  0 09:11 pts/0    00:00:00 /bin/sh bin/mysqld_safe --user=mysql
mysql    28346 28276  0 09:11 pts/0    00:00:00 /usr/local/mysql-5.7.41-linux-glibc2.12-x86_64/bin/mysqld --basedir=/usr/local/mysql-5.7.41-linux-glibc2.12-x86_64 --datadir=/usr/local/mysql-5.7.41-linux-glibc2.12-x86_64/data --plugin-dir=/usr/local/mysql-5.7.41-linux-glibc2.12-x86_64/lib/plugin --user=mysql --log-error=centos-mysql.err --pid-file=centos-mysql.pid
vagrant  28376 28073  0 09:13 pts/0    00:00:00 grep --color=auto mysqld
```

方法二：使用centos6 mysql.server (system V 脚本)
这种方式本质上还是使用的 mysqld_safe

```shell
cp support-files/mysql.server /etc/init.d/mysqld
/etc/init.d/mysqld start 

chkconfig --add /etc/init.d/mysqld
chkconfig mysqld on
service mysqld restart
```

或者

```shell
/etc/init.d/mysqld start
```

## 配置环境变量

为了避免在客户端使用mysql可执行程序时输入完整的路径，可以将mysql的二进制程序路径添加到 PATH 变量中：

```shell
echo 'export PATH=$PATH:/usr/local/mysql/bin' >> ~/.bash_profile
source ~/.bash_profile
```

## 连接MySQL

1.登录

```shell
$ mysql -uroot -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.41

Copyright (c) 2000, 2023, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

2.修改密码

```shell
mysql> show databases;
ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.
mysql> alter user 'root'@'localhost' identified by '123';
Query OK, 0 rows affected (0.00 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.00 sec)

mysql> grant all privileges on *.* to 'root'@'%' identified by '123';
Query OK, 0 rows affected, 1 warning (0.00 sec)


mysql> grant all privileges on *.* to 'root'@'%' identified by '123';
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> exit
Bye
```

## 配置 error log

```shell
[client]
port = 3306
socket = /usr/local/mysql/data/mysql.sock

[mysqld]
basedir = /usr/local/mysql
datadir = /usr/local/mysql/data
socket = /usr/local/mysql/data/mysql.sock
# symbolic-links=0

[mysqld_safe]
log-error=/usr/local/mysql/logs/mysqld.log
pid-file=/usr/local/mysql/data/mysqld.pid
```

重启mysql

```shell
[root@localhost mysql]# service mysqld restart
 ERROR! MySQL server PID file could not be found!
Starting MySQL.2023-06-18T14:32:00.501051Z mysqld_safe error: log-error set to '/usr/local/mysql/logs/mysqld.log', however file don't exists. Create writable for user 'mysql'.
 ERROR! The server quit without updating PID file (/usr/local/mysql/data/localhost.localdomain.pid).
[root@localhost mysql]#
```

重新初始化：

```shell
[root@localhost mysql]# bin/mysqld --initialize --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data --log-error=/usr/local/mysql/logs/mysqld.log --pid-file=/usr/local/mysql/data/mysqld.pid
2023-06-18T14:32:52.010175Z 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2023-06-18T14:32:52.012477Z 0 [ERROR] --initialize specified but the data directory has files in it. Aborting.
2023-06-18T14:32:52.012521Z 0 [ERROR] Aborting

[root@localhost mysql]# mv data/ data.bak
[root@localhost mysql]#
[root@localhost mysql]# bin/mysqld --initialize --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data --log-error=/usr/local/mysql/logs/mysqld.log --pid-file=/usr/local/mysql/data/mysqld.pid
[root@localhost mysql]# bin/mysql_ssl_rsa_setup --datadir=/usr/local/mysql/data

```

重启：

```shell
[root@localhost mysql]# service mysqld start
Starting MySQL. SUCCESS!
[root@localhost mysql]#
```

连接登录发现之前的账号密码已经无法登录：
```shell
[root@localhost mysql]# mysql -u root -p
Enter password:
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES)
[root@localhost mysql]#
```

## 允许远程登录

```sql
grant all priviletes on *.* to 'root'@'%' identified by '123' with grant option;
```

## socket 连接

```shell
mysql -uroot -p'123' -S /usr/local/mysql/data/mysql.sock 
```

## 直接修改密码

```shell
mysqladmin -uroot -p'old password' password "new password"
```

## 其他操作

```shell
$ sudo mkdir {data,logs}
$ sudo chown mysql:mysql mysql-files data logs
```

```shell
# bin/mysqld --initialize --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data --log-error=/usr/local/mysql/logs/mysqld.log --pid-file=/usr/local/mysql/data/mysqld.pid
```

```shell
[client]
port = 3306
socket = /usr/local/mysql/data/mysql.sock

[mysqld]
character_set_server=utf8mb4
basedir = /usr/local/mysql
datadir = /usr/local/mysql/data
port = 3306
socket = /usr/local/mysql/data/mysql.sock
log-error=/usr/local/mysql/logs/mysqld.log
pid-file=/usr/local/mysql/data/mysqld.pid
```

```shell
# cp support-files/my-default.cnf  /etc/my.cnf
cat  > /etc/my.cnf <<EOF
[client]
port = 3306
socket = /usr/local/mysql/data/mysql.sock

[mysqld]
character_set_server=utf8mb4
skip_name_resolve=1
innodb_file_per_table=1
max_connections=20000
basedir = /usr/local/mysql/
datadir = /usr/local/mysql/data
port = 3306
socket = /usr/local/mysql/data/mysql.sock
log-error=/usr/local/mysql/logs/mysqld.log
pid-file=/usr/local/mysql/data/mysqld.pid
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES 
EOF

cp support-files/mysql.server /etc/init.d/mysqld
```
参考：https://www.cnblogs.com/kntvops/p/17305783.html

## 加入系统启动服务

另外还可以使用 `mysql.server 启动脚本` 的方式启动数据库，在 `support-files` 目录下，可以把这个脚本加到系统自启动目录下，然后使用 `chkconfig` 激活

```shell
$ sudo cp support-files/mysql.server /etc/init.d/mysql.server
```

有时候会看到别人会将文件加入到/etc/rc.d/init.d，如：

```shell
$ sudo cp support-files/mysql.server /etc/rc.d/init.d/mysqld
$ sudo chmod +x /etc/rc.d/init.d/mysqld
```

/etc/init.d 与 /etc/rc.d/init.d 的区别：
/etc/init.d 与 /etc/rc.d/init.d 都用来存放服务脚本，Linux启动时会自动在这些目录下寻找服务脚本，并根据脚本的运行级别确定不同服务的启动级别

CentOS 下 /etc/init.d 是 /etc/rc.d/init.d 的软链，而 Ubuntu下是没有/etc/rc.d/init.d目录的，为了保持同一个服务在不同的操作系统间的统一，所以会将服务都放到 /etc/init.d 目录下，最终达成的效果都是相同的。

在CentOS和Ubuntu中，除了服务脚本放置的目录是相同的，服务脚本的编写及服务配置都是不同的。比如CentOS使用Chkconfig进行配置，而Ubuntu使用sysv-rc-conf进行配置。

参考：https://blog.csdn.net/yongche_shi/article/details/44488815


## 将mysql.server服务加入到系统服务

1.添加服务

```shell
$ sudo chkconfig --add mysql.server
$ sudo chkconfig --list mysql.server

Note: This output shows SysV services only and does not include native
      systemd services. SysV configuration data might be overridden by native
      systemd configuration.

      If you want to list systemd services use 'systemctl list-unit-files'.
      To see services enabled on particular target use
      'systemctl list-dependencies [target]'.

mysql.server   	0:off	1:off	2:on	3:on	4:on	5:on	6:off
```

2.启动服务

```shell
$ sudo service mysql.server status
 ERROR! MySQL is not running
$ sudo service mysql.server start
Starting MySQL. SUCCESS!
$ sudo service mysql.server restart
Shutting down MySQL..2023-05-02T01:24:26.252061Z mysqld_safe mysqld from pid file /usr/local/mysql-5.7.41-linux-glibc2.12-x86_64/data/centos-mysql.pid ended
 SUCCESS!
Starting MySQL. SUCCESS!
[1]+  Done                    sudo bin/mysqld_safe --user=mysql
$ sudo service mysql.server status
 SUCCESS! MySQL running (28346)
$ sudo service mysql.server stop
Shutting down MySQL.. SUCCESS!
```

## 设置密码校验策略

为了开发方便可以设置下密码策略，但是生产环境强力不推荐
```shell
set global validate_password_policy=LOW
set global validate_password_length=4;
set password=password('123');
```

打开 3306 端口
```shell
firewall-cmd --zone=public --add-port=3306/tcp permanent
firewall-cmd reload
```
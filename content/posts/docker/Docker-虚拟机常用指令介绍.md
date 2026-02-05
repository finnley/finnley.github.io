+++
title = 'Docker常用指令总结'
date = 2018-09-23T00:19:16+08:00
draft = true
categories = [ "Docker" ]
tags = [ "docker" ]
+++

![](/images/docker/docker-overview.jpg)

```shell
docker search {Image Name}
```



### 暂停容器

```
docker pause myjava
```

### 恢复容器

```
docker unpause myjava
```

### 彻底停止

```
docker stop myjava
```

### 恢复运行

```
docker start -i myjava
```

之前我们在容器的交互界面使用 `exit` 退出容器，该命令不仅是退出容器，还停止运行了，使容器进入到 `stop` 状态里面，如果要运行执行容器的话就必须使用 `start` 命令去重新启动容器。

* 重新启动刚才关闭的容器

![](https://images.notes.xuepincat.com/docker/22.png)

* 重新打开一个终端，并连接到linux上，在这里面将 `myjava` 的容器暂停一下

![](https://images.notes.xuepincat.com/docker/23.png)

* 恢复容器

![](https://images.notes.xuepincat.com/docker/24.png)

* 如果想删除容器，前提是必须彻底停止容器，然后再去删除容器

![](https://images.notes.xuepincat.com/docker/25.png)

* 查看容器

![](https://images.notes.xuepincat.com/docker/26.png)

## Docker虚拟机常用命令

1. 先更新软件包

   ```shell
   yum -y update
   ```

2. 安装Docker虚拟机

   ```shell
   yum install -y docker
   ```

3. 运行、重启、关闭Docker虚拟机

   ```shell
   service docker start
   service docker start
   service docker stop
   ```

4. 搜索镜像

   ```shell
   docker search 镜像名称
   ```

5. 下载镜像

   ```shell
   docker pull 镜像名称
   ```

6. 查看镜像

   ```shell
   docker images
   ```

7. 删除镜像

   ```shell
   docker rmi 镜像名称
   ```

8. 运行容器

   ```shell
   docker run 启动参数  镜像名称
   ```

9. 查看容器列表

   ```shell
   docker ps -a
   ```

10. 停止、挂起、恢复容器

   ```shell
   docker stop 容器ID
   docker pause 容器ID
   docker unpase 容器ID
   ```

11. 查看容器信息

    ```shell
    docker inspect 容器ID
    ```

12. 删除容器

    ```shell
    docker rm 容器ID
    ```

13. 数据卷管理

    ```shell
    docker volume create 数据卷名称  #创建数据卷
    docker volume rm 数据卷名称  #删除数据卷
    docker volume inspect 数据卷名称  #查看数据卷
    ```

14. 网络管理

    ```shell
    docker network ls 查看网络信息
    docker network create --subnet=网段 网络名称
    docker network rm 网络名称
    ```

15. 避免VM虚拟机挂起恢复之后，Docker虚拟机断网

    ```shell
    vi /etc/sysctl.conf
    ```


    文件中添加`net.ipv4.ip_forward=1`这个配置

    ​```shell
    #重启网络服务
    systemctl  restart network
    ​```

## 安装PXC集群，负载均衡，双机热备

1. 安装PXC镜像

   ```shell
   docker pull percona/percona-xtradb-cluster
   ```

2. 为PXC镜像改名

   ```shell
   docker tag percona/percona-xtradb-cluster pxc
   ```

3. 创建net1网段

   ```shell
   docker network create --subnet=172.18.0.0/24 net1
   ```

4. 创建5个数据卷

   ```shell
   docker volume create --name v1
   docker volume create --name v2
   docker volume create --name v3
   docker volume create --name v4
   docker volume create --name v5
   ```

5. 创建备份数据卷（用于热备份数据）

   ```shell
   docker volume create --name backup
   ```

6. 创建5节点的PXC集群

   注意，每个MySQL容器创建之后，因为要执行PXC的初始化和加入集群等工作，耐心等待1分钟左右再用客户端连接MySQL。另外，必须第1个MySQL节点启动成功，用MySQL客户端能连接上之后，再去创建其他MySQL节点。

   ```shell
   #创建第1个MySQL节点
   docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=abc123456 -e CLUSTER_NAME=PXC -e XTRABACKUP_PASSWORD=abc123456 -v v1:/var/lib/mysql -v backup:/data --privileged --name=node1 --net=net1 --ip 172.18.0.2 pxc
   #创建第2个MySQL节点
   docker run -d -p 3307:3306 -e MYSQL_ROOT_PASSWORD=abc123456 -e CLUSTER_NAME=PXC -e XTRABACKUP_PASSWORD=abc123456 -e CLUSTER_JOIN=node1 -v v2:/var/lib/mysql -v backup:/data --privileged --name=node2 --net=net1 --ip 172.18.0.3 pxc
   #创建第3个MySQL节点
   docker run -d -p 3308:3306 -e MYSQL_ROOT_PASSWORD=abc123456 -e CLUSTER_NAME=PXC -e XTRABACKUP_PASSWORD=abc123456 -e CLUSTER_JOIN=node1 -v v3:/var/lib/mysql --privileged --name=node3 --net=net1 --ip 172.18.0.4 pxc
   #创建第4个MySQL节点
   docker run -d -p 3309:3306 -e MYSQL_ROOT_PASSWORD=abc123456 -e CLUSTER_NAME=PXC -e XTRABACKUP_PASSWORD=abc123456 -e CLUSTER_JOIN=node1 -v v4:/var/lib/mysql --privileged --name=node4 --net=net1 --ip 172.18.0.5 pxc
   #创建第5个MySQL节点
   docker run -d -p 3310:3306 -e MYSQL_ROOT_PASSWORD=abc123456 -e CLUSTER_NAME=PXC -e XTRABACKUP_PASSWORD=abc123456 -e CLUSTER_JOIN=node1 -v v5:/var/lib/mysql -v backup:/data --privileged --name=node5 --net=net1 --ip 172.18.0.6 pxc
   ```

7. 安装Haproxy镜像

   ```shell
   docker pull haproxy
   ```

8. 宿主机上编写Haproxy配置文件

   ```shell
   vi /home/soft/haproxy.cfg
   ```

   配置文件如下：

   ```properties
   global
   	#工作目录
   	chroot /usr/local/etc/haproxy
   	#日志文件，使用rsyslog服务中local5日志设备（/var/log/local5），等级info
   	log 127.0.0.1 local5 info
   	#守护进程运行
   	daemon

   defaults
   	log	global
   	mode	http
   	#日志格式
   	option	httplog
   	#日志中不记录负载均衡的心跳检测记录
   	option	dontlognull
       #连接超时（毫秒）
   	timeout connect 5000
       #客户端超时（毫秒）
   	timeout client  50000
   	#服务器超时（毫秒）
       timeout server  50000

   #监控界面	
   listen  admin_stats
   	#监控界面的访问的IP和端口
   	bind  0.0.0.0:8888
   	#访问协议
       mode        http
   	#URI相对地址
       stats uri   /dbs
   	#统计报告格式
       stats realm     Global\ statistics
   	#登陆帐户信息
       stats auth  admin:abc123456
   #数据库负载均衡
   listen  proxy-mysql
   	#访问的IP和端口
   	bind  0.0.0.0:3306  
       #网络协议
   	mode  tcp
   	#负载均衡算法（轮询算法）
   	#轮询算法：roundrobin
   	#权重算法：static-rr
   	#最少连接算法：leastconn
   	#请求源IP算法：source 
       balance  roundrobin
   	#日志格式
       option  tcplog
   	#在MySQL中创建一个没有权限的haproxy用户，密码为空。Haproxy使用这个账户对MySQL数据库心跳检测
       option  mysql-check user haproxy
       server  MySQL_1 172.18.0.2:3306 check weight 1 maxconn 2000  
       server  MySQL_2 172.18.0.3:3306 check weight 1 maxconn 2000  
   	server  MySQL_3 172.18.0.4:3306 check weight 1 maxconn 2000 
   	server  MySQL_4 172.18.0.5:3306 check weight 1 maxconn 2000
   	server  MySQL_5 172.18.0.6:3306 check weight 1 maxconn 2000
   	#使用keepalive检测死链
       option  tcpka  
   ```

9. 创建两个Haproxy容器

   ```shell
   #创建第1个Haproxy负载均衡服务器
   docker run -it -d -p 4001:8888 -p 4002:3306 -v /home/soft/haproxy:/usr/local/etc/haproxy --name h1 --privileged --net=net1 --ip 172.18.0.7 haproxy
   #进入h1容器，启动Haproxy
   docker exec -it h1 bash
   haproxy -f /usr/local/etc/haproxy/haproxy.cfg
   #创建第2个Haproxy负载均衡服务器
   docker run -it -d -p 4003:8888 -p 4004:3306 -v /home/soft/haproxy:/usr/local/etc/haproxy --name h2 --privileged --net=net1 --ip 172.18.0.8 haproxy
   #进入h2容器，启动Haproxy
   docker exec -it h2 bash
   haproxy -f /usr/local/etc/haproxy/haproxy.cfg
   ```

10. Haproxy容器内安装Keepalived，设置虚拟IP

    ```shell
    #进入h1容器
    docker exec -it h1 bash
    #更新软件包
    apt-get update
    #安装VIM
    apt-get install vim
    #安装Keepalived
    apt-get install keepalived
    #编辑Keepalived配置文件（参考下方配置文件）
    vim /etc/keepalived/keepalived.conf
    #启动Keepalived
    service keepalived start
    #宿主机执行ping命令
    ping 172.18.0.201
    ```

    配置文件内容如下：

    ```
    vrrp_instance  VI_1 {
        state  MASTER
        interface  eth0
        virtual_router_id  51
        priority  100
        advert_int  1
        authentication {
            auth_type  PASS
            auth_pass  123456
        }
        virtual_ipaddress {
            172.18.0.201
        }
    }
    ```

    ```shell
    #进入h2容器
    docker exec -it h2 bash
    #更新软件包
    apt-get update
    #安装VIM
    apt-get install vim
    #安装Keepalived
    apt-get install keepalived
    #编辑Keepalived配置文件
    vim /etc/keepalived/keepalived.conf
    #启动Keepalived
    service keepalived start
    #宿主机执行ping命令
    ping 172.18.0.201
    ```

    配置文件内容如下：

    ```shell
    vrrp_instance  VI_1 {
        state  MASTER
        interface  eth0
        virtual_router_id  51
        priority  100
        advert_int  1
        authentication {
            auth_type  PASS
            auth_pass  123456
        }
        virtual_ipaddress {
            172.18.0.201
        }
    }
    ```

11. 宿主机安装Keepalived，实现双击热备

    ```shell
    #宿主机执行安装Keepalived
    yum -y install keepalived
    #修改Keepalived配置文件
    vi /etc/keepalived/keepalived.conf
    #启动Keepalived
    service keepalived start
    ```

    Keepalived配置文件如下：

    ```shell
    vrrp_instance VI_1 {
        state MASTER
        interface ens33
        virtual_router_id 51
        priority 100
        advert_int 1
        authentication {
            auth_type PASS
            auth_pass 1111
        }
        virtual_ipaddress {
           	192.168.99.150
        }
    }

    virtual_server 192.168.99.150 8888 {
        delay_loop 3
        lb_algo rr 
        lb_kind NAT
        persistence_timeout 50
        protocol TCP

        real_server 172.18.0.201 8888 {
            weight 1
        }
    }

    virtual_server 192.168.99.150 3306 {
        delay_loop 3
        lb_algo rr 
        lb_kind NAT
        persistence_timeout 50
        protocol TCP

        real_server 172.18.0.201 3306 {
            weight 1
        }
    }
    ```

12. 热备份数据

    ```shell
    #进入node1容器
    docker exec -it node1 bash
    #更新软件包
    apt-get update
    #安装热备工具
    apt-get install percona-xtrabackup-24
    #全量热备
    innobackupex --user=root --password=abc123456 /data/backup/full
    ```

13. 冷还原数据
    停止其余4个节点，并删除节点

    ```shell
    docker stop node2
    docker stop node3
    docker stop node4
    docker stop node5
    docker rm node2
    docker rm node3
    docker rm node4
    docker rm node5
    ```

    node1容器中删除MySQL的数据

    ```shell
    #删除数据
    rm -rf /var/lib/mysql/*
    #清空事务
    innobackupex --user=root --password=abc123456 --apply-back /data/backup/full/2018-04-15_05-09-07/
    #还原数据
    innobackupex --user=root --password=abc123456 --copy-back  /data/backup/full/2018-04-15_05-09-07/
    ```

    重新创建其余4个节点，组件PXC集群

## 安装Redis，配置RedisCluster集群

1. 安装Redis镜像

   ```shell
   docker pull yyyyttttwwww/redis
   ```

2. 创建net2网段

   ```shell
   docker network create --subnet=172.19.0.0/16 net2
   ```

3. 创建6节点Redis容器

   ```shell
   docker run -it -d --name r1 -p 5001:6379 --net=net2 --ip 172.19.0.2 redis bash
   docker run -it -d --name r2 -p 5002:6379 --net=net2 --ip 172.19.0.3 redis bash
   docker run -it -d --name r3 -p 5003:6379 --net=net2 --ip 172.19.0.4 redis bash
   docker run -it -d --name r4 -p 5004:6379 --net=net2 --ip 172.19.0.5 redis bash
   docker run -it -d --name r5 -p 5005:6379 --net=net2 --ip 172.19.0.6 redis bash
   ```

4. 启动6节点Redis服务器

   ```shell
   #进入r1节点
   docker exec -it r1 bash
   cp /home/redis/redis.conf /usr/redis/redis.conf
   cd /usr/redis/src
   ./redis-server ../redis.conf
   #进入r2节点
   docker exec -it r2 bash
   cp /home/redis/redis.conf /usr/redis/redis.conf
   cd /usr/redis/src
   ./redis-server ../redis.conf
   #进入r3节点
   docker exec -it r3 bash
   cp /home/redis/redis.conf /usr/redis/redis.conf
   cd /usr/redis/src
   ./redis-server ../redis.conf
   #进入r4节点
   docker exec -it r4 bash
   cp /home/redis/redis.conf /usr/redis/redis.conf
   cd /usr/redis/src
   ./redis-server ../redis.conf
   #进入r5节点
   docker exec -it r5 bash
   cp /home/redis/redis.conf /usr/redis/redis.conf
   cd /usr/redis/src
   ./redis-server ../redis.conf
   #进入r6节点
   docker exec -it r6 bash
   cp /home/redis/redis.conf /usr/redis/redis.conf
   cd /usr/redis/src
   ./redis-server ../redis.conf
   ```

5. 创建Cluster集群

   ```shell
   #在r1节点上执行下面的指令
   cd /usr/redis/src
   mkdir -p ../cluster
   cp redis-trib.rb ../cluster/
   cd ../cluster
   #创建Cluster集群
   ./redis-trib.rb create --replicas 1 172.19.0.2:6379 172.19.0.3:6379 172.19.0.4:6379 172.19.0.5:6379 172.19.0.6:6379 172.19.0.7:6379
   ```



## 打包部署后端项目

1. 进入人人开源后端项目，执行打包（修改配置文件，更改端口，打包三次生成三个JAR文件）

   ```shell
   mvn clean install -Dmaven.test.skip=true
   ```

2. 安装Java镜像

   ```shell
   docker pull java
   ```

3. 创建3节点Java容器

   ```shell
   #创建数据卷，上传JAR文件
   docker volume create j1
   #启动容器
   docker run -it -d --name j1 -v j1:/home/soft --net=host java
   #进入j1容器
   docker exec -it j1 bash
   #启动Java项目
   nohup java -jar /home/soft/renren-fast.jar

   #创建数据卷，上传JAR文件
   docker volume create j2
   #启动容器
   docker run -it -d --name j2 -v j2:/home/soft --net=host java
   #进入j1容器
   docker exec -it j2 bash
   #启动Java项目
   nohup java -jar /home/soft/renren-fast.jar

   #创建数据卷，上传JAR文件
   docker volume create j3
   #启动容器
   docker run -it -d --name j3 -v j3:/home/soft --net=host java
   #进入j1容器
   docker exec -it j3 bash
   #启动Java项目
   nohup java -jar /home/soft/renren-fast.jar
   ```

4. 安装Nginx镜像

   ```shell
   docker pull nginx
   ```

5. 创建Nginx容器，配置负载均衡

   宿主机上/home/n1/nginx.conf配置文件内容如下：

   ```properties
   user  nginx;
   worker_processes  1;
   error_log  /var/log/nginx/error.log warn;
   pid        /var/run/nginx.pid;

   events {
       worker_connections  1024;
   }

   http {
       include       /etc/nginx/mime.types;
       default_type  application/octet-stream;

       log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                         '$status $body_bytes_sent "$http_referer" '
                         '"$http_user_agent" "$http_x_forwarded_for"';

       access_log  /var/log/nginx/access.log  main;

       sendfile        on;
       #tcp_nopush     on;

       keepalive_timeout  65;

       #gzip  on;
   	
   	proxy_redirect          off;
   	proxy_set_header        Host $host;
   	proxy_set_header        X-Real-IP $remote_addr;
   	proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
   	client_max_body_size    10m;
   	client_body_buffer_size   128k;
   	proxy_connect_timeout   5s;
   	proxy_send_timeout      5s;
   	proxy_read_timeout      5s;
   	proxy_buffer_size        4k;
   	proxy_buffers           4 32k;
   	proxy_busy_buffers_size  64k;
   	proxy_temp_file_write_size 64k;
   	
   	upstream tomcat {
   		server 192.168.99.104:6001;
   		server 192.168.99.104:6002;
   		server 192.168.99.104:6003;
   	}
   	server {
           listen       6101;
           server_name  192.168.99.104; 
           location / {  
               proxy_pass   http://tomcat;
               index  index.html index.htm;  
           }  
       }
   }
   ```

   创建第1个Nginx节点

   ```shell
   docker run -it -d --name n1 -v /home/n1/nginx.conf:/etc/nginx/nginx.conf --net=host --privileged nginx

   ```

   宿主机上/home/n2/nginx.conf配置文件内容如下：

   ```properties
   user  nginx;
   worker_processes  1;
   error_log  /var/log/nginx/error.log warn;
   pid        /var/run/nginx.pid;

   events {
       worker_connections  1024;
   }

   http {
       include       /etc/nginx/mime.types;
       default_type  application/octet-stream;

       log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                         '$status $body_bytes_sent "$http_referer" '
                         '"$http_user_agent" "$http_x_forwarded_for"';

       access_log  /var/log/nginx/access.log  main;

       sendfile        on;
       #tcp_nopush     on;

       keepalive_timeout  65;

       #gzip  on;
   	
   	proxy_redirect          off;
   	proxy_set_header        Host $host;
   	proxy_set_header        X-Real-IP $remote_addr;
   	proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
   	client_max_body_size    10m;
   	client_body_buffer_size   128k;
   	proxy_connect_timeout   5s;
   	proxy_send_timeout      5s;
   	proxy_read_timeout      5s;
   	proxy_buffer_size        4k;
   	proxy_buffers           4 32k;
   	proxy_busy_buffers_size  64k;
   	proxy_temp_file_write_size 64k;
   	
   	upstream tomcat {
   		server 192.168.99.104:6001;
   		server 192.168.99.104:6002;
   		server 192.168.99.104:6003;
   	}
   	server {
           listen       6102;
           server_name  192.168.99.104; 
           location / {  
               proxy_pass   http://tomcat;
               index  index.html index.htm;  
           }  
       }
   }
   ```

   创建第2个Nginx节点

   ```shell
   docker run -it -d --name n2 -v /home/n2/nginx.conf:/etc/nginx/nginx.conf --net=host --privileged nginx
   ```

6. 在Nginx容器安装Keepalived

   ```shell
   #进入n1节点
   docker exec -it n1 bash
   #更新软件包
   apt-get update
   #安装VIM
   apt-get install vim
   #安装Keepalived
   apt-get install keepalived
   #编辑Keepalived配置文件(如下)
   vim /etc/keepalived/keepalived.conf
   #启动Keepalived
   service keepalived start
   ```

   ```
   vrrp_instance VI_1 {
       state MASTER
       interface ens33
       virtual_router_id 51
       priority 100
       advert_int 1
       authentication {
           auth_type PASS
           auth_pass 123456
       }
       virtual_ipaddress {
           192.168.99.151
       }
   }
   virtual_server 192.168.99.151 6201 {
       delay_loop 3
       lb_algo rr
       lb_kind NAT
       persistence_timeout 50
       protocol TCP
       real_server 192.168.99.104 6101 {
           weight 1
       }
   }
   ```

   ```shell
   #进入n1节点
   docker exec -it n2 bash
   #更新软件包
   apt-get update
   #安装VIM
   apt-get install vim
   #安装Keepalived
   apt-get install keepalived
   #编辑Keepalived配置文件(如下)
   vim /etc/keepalived/keepalived.conf
   #启动Keepalived
   service keepalived start
   ```

   ```shell
   vrrp_instance VI_1 {
       state MASTER
       interface ens33
       virtual_router_id 51
       priority 100
       advert_int 1
       authentication {
           auth_type PASS
           auth_pass 123456
       }
       virtual_ipaddress {
           192.168.99.151
       }
   }
   virtual_server 192.168.99.151 6201 {
       delay_loop 3
       lb_algo rr
       lb_kind NAT
       persistence_timeout 50
       protocol TCP
       real_server 192.168.99.104 6102 {
           weight 1
       }
   }
   ```



## 打包部署后端项目

1. 在前端项目路径下执行打包指令

   ```shell
   npm run build
   ```

2. build目录的文件拷贝到宿主机的/home/fn1/renren-vue、/home/fn2/renren-vue、/home/fn3/renren-vue的目录下面

3. 创建3节点的Nginx，部署前端项目

   宿主机/home/fn1/nginx.conf的配置文件

   ```
   user  nginx;
   worker_processes  1;
   error_log  /var/log/nginx/error.log warn;
   pid        /var/run/nginx.pid;

   events {
       worker_connections  1024;
   }

   http {
       include       /etc/nginx/mime.types;
       default_type  application/octet-stream;

       log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                         '$status $body_bytes_sent "$http_referer" '
                         '"$http_user_agent" "$http_x_forwarded_for"';

       access_log  /var/log/nginx/access.log  main;

       sendfile        on;
       #tcp_nopush     on;

       keepalive_timeout  65;

       #gzip  on;
   	
   	proxy_redirect          off;
   	proxy_set_header        Host $host;
   	proxy_set_header        X-Real-IP $remote_addr;
   	proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
   	client_max_body_size    10m;
   	client_body_buffer_size   128k;
   	proxy_connect_timeout   5s;
   	proxy_send_timeout      5s;
   	proxy_read_timeout      5s;
   	proxy_buffer_size        4k;
   	proxy_buffers           4 32k;
   	proxy_busy_buffers_size  64k;
   	proxy_temp_file_write_size 64k;
   	
   	server {
   		listen 6501;
   		server_name  192.168.99.104;
   		location  /  {
   			root  /home/fn1/renren-vue;
   			index  index.html;
   		}
   	}
   }
   ```

   ```shell
   #启动第fn1节点
   docker run -it -d --name fn1 -v /home/fn1/nginx.conf:/etc/nginx/nginx.conf -v /home/fn1/renren-vue:/home/fn1/renren-vue --privileged --net=host nginx
   ```

   宿主机/home/fn2/nginx.conf的配置文件

   ```shell
   user  nginx;
   worker_processes  1;
   error_log  /var/log/nginx/error.log warn;
   pid        /var/run/nginx.pid;

   events {
       worker_connections  1024;
   }

   http {
       include       /etc/nginx/mime.types;
       default_type  application/octet-stream;

       log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                         '$status $body_bytes_sent "$http_referer" '
                         '"$http_user_agent" "$http_x_forwarded_for"';

       access_log  /var/log/nginx/access.log  main;

       sendfile        on;
       #tcp_nopush     on;

       keepalive_timeout  65;

       #gzip  on;
   	
   	proxy_redirect          off;
   	proxy_set_header        Host $host;
   	proxy_set_header        X-Real-IP $remote_addr;
   	proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
   	client_max_body_size    10m;
   	client_body_buffer_size   128k;
   	proxy_connect_timeout   5s;
   	proxy_send_timeout      5s;
   	proxy_read_timeout      5s;
   	proxy_buffer_size        4k;
   	proxy_buffers           4 32k;
   	proxy_busy_buffers_size  64k;
   	proxy_temp_file_write_size 64k;
   	
   	server {
   		listen 6502;
   		server_name  192.168.99.104;
   		location  /  {
   			root  /home/fn2/renren-vue;
   			index  index.html;
   		}
   	}
   }
   ```

   ```shell
   #启动第fn2节点
   docker run -it -d --name fn2 -v /home/fn2/nginx.conf:/etc/nginx/nginx.conf -v /home/fn2/renren-vue:/home/fn2/renren-vue --privileged --net=host nginx
   ```

   宿主机/home/fn3/nginx.conf的配置文件

   ```shell
   user  nginx;
   worker_processes  1;
   error_log  /var/log/nginx/error.log warn;
   pid        /var/run/nginx.pid;

   events {
       worker_connections  1024;
   }

   http {
       include       /etc/nginx/mime.types;
       default_type  application/octet-stream;

       log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                         '$status $body_bytes_sent "$http_referer" '
                         '"$http_user_agent" "$http_x_forwarded_for"';

       access_log  /var/log/nginx/access.log  main;

       sendfile        on;
       #tcp_nopush     on;

       keepalive_timeout  65;

       #gzip  on;
   	
   	proxy_redirect          off;
   	proxy_set_header        Host $host;
   	proxy_set_header        X-Real-IP $remote_addr;
   	proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
   	client_max_body_size    10m;
   	client_body_buffer_size   128k;
   	proxy_connect_timeout   5s;
   	proxy_send_timeout      5s;
   	proxy_read_timeout      5s;
   	proxy_buffer_size        4k;
   	proxy_buffers           4 32k;
   	proxy_busy_buffers_size  64k;
   	proxy_temp_file_write_size 64k;
   	
   	server {
   		listen 6503;
   		server_name  192.168.99.104;
   		location  /  {
   			root  /home/fn3/renren-vue;
   			index  index.html;
   		}
   	}
   }
   ```

   启动fn3节点

   ```shell
   #启动第fn3节点
   docker run -it -d --name fn3 -v /home/fn3/nginx.conf:/etc/nginx/nginx.conf -v /home/fn3/renren-vue:/home/fn3/renren-vue --privileged --net=host nginx
   ```

4. 配置负载均衡

   宿主机/home/ff1/nginx.conf配置文件

   ```shell
   user  nginx;
   worker_processes  1;
   error_log  /var/log/nginx/error.log warn;
   pid        /var/run/nginx.pid;

   events {
       worker_connections  1024;
   }

   http {
       include       /etc/nginx/mime.types;
       default_type  application/octet-stream;

       log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                         '$status $body_bytes_sent "$http_referer" '
                         '"$http_user_agent" "$http_x_forwarded_for"';

       access_log  /var/log/nginx/access.log  main;

       sendfile        on;
       #tcp_nopush     on;

       keepalive_timeout  65;

       #gzip  on;
   	
   	proxy_redirect          off;
   	proxy_set_header        Host $host;
   	proxy_set_header        X-Real-IP $remote_addr;
   	proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
   	client_max_body_size    10m;
   	client_body_buffer_size   128k;
   	proxy_connect_timeout   5s;
   	proxy_send_timeout      5s;
   	proxy_read_timeout      5s;
   	proxy_buffer_size        4k;
   	proxy_buffers           4 32k;
   	proxy_busy_buffers_size  64k;
   	proxy_temp_file_write_size 64k;
   	
   	upstream fn {
   		server 192.168.99.104:6501;
   		server 192.168.99.104:6502;
   		server 192.168.99.104:6503;
   	}
   	server {
           listen       6601;
           server_name  192.168.99.104; 
           location / {  
               proxy_pass   http://fn;
               index  index.html index.htm;  
           }  
       }
   }
   ```

   ```shell
   #启动ff1节点
   docker run -it -d --name ff1 -v /home/ff1/nginx.conf:/etc/nginx/nginx.conf --net=host --privileged nginx
   ```

   宿主机/home/ff2/nginx.conf配置文件

   ```shell
   user  nginx;
   worker_processes  1;
   error_log  /var/log/nginx/error.log warn;
   pid        /var/run/nginx.pid;

   events {
       worker_connections  1024;
   }

   http {
       include       /etc/nginx/mime.types;
       default_type  application/octet-stream;

       log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                         '$status $body_bytes_sent "$http_referer" '
                         '"$http_user_agent" "$http_x_forwarded_for"';

       access_log  /var/log/nginx/access.log  main;

       sendfile        on;
       #tcp_nopush     on;

       keepalive_timeout  65;

       #gzip  on;
   	
   	proxy_redirect          off;
   	proxy_set_header        Host $host;
   	proxy_set_header        X-Real-IP $remote_addr;
   	proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
   	client_max_body_size    10m;
   	client_body_buffer_size   128k;
   	proxy_connect_timeout   5s;
   	proxy_send_timeout      5s;
   	proxy_read_timeout      5s;
   	proxy_buffer_size        4k;
   	proxy_buffers           4 32k;
   	proxy_busy_buffers_size  64k;
   	proxy_temp_file_write_size 64k;
   	
   	upstream fn {
   		server 192.168.99.104:6501;
   		server 192.168.99.104:6502;
   		server 192.168.99.104:6503;
   	}
   	server {
           listen       6602;
           server_name  192.168.99.104; 
           location / {  
               proxy_pass   http://fn;
               index  index.html index.htm;  
           }  
       }
   }
   ```

   ```shell
   #启动ff2节点
   docker run -it -d --name ff2 -v /home/ff2/nginx.conf:/etc/nginx/nginx.conf --net=host --privileged nginx
   ```

5. 配置双机热备

   ```shell
   #进入ff1节点
   docker exec -it ff1 bash
   #更新软件包
   apt-get update
   #安装VIM
   apt-get install vim
   #安装Keepalived
   apt-get install keepalived
   #编辑Keepalived配置文件(如下)
   vim /etc/keepalived/keepalived.conf
   #启动Keepalived
   service keepalived start
   ```

   ```shell
   vrrp_instance VI_1 {
       state MASTER
       interface ens33
       virtual_router_id 52
       priority 100
       advert_int 1
       authentication {
           auth_type PASS
           auth_pass 123456
       }
       virtual_ipaddress {
           192.168.99.152
       }
   }
   virtual_server 192.168.99.151 6701 {
       delay_loop 3
       lb_algo rr
       lb_kind NAT
       persistence_timeout 50
       protocol TCP
       real_server 192.168.99.104 6601 {
           weight 1
       }
   }
   ```

   ```shell
   #进入ff1节点
   docker exec -it ff2 bash
   #更新软件包
   apt-get update
   #安装VIM
   apt-get install vim
   #安装Keepalived
   apt-get install keepalived
   #编辑Keepalived配置文件(如下)
   vim /etc/keepalived/keepalived.conf
   #启动Keepalived
   service keepalived start
   ```

   ```shell
   vrrp_instance VI_1 {
       state MASTER
       interface ens33
       virtual_router_id 52
       priority 100
       advert_int 1
       authentication {
           auth_type PASS
           auth_pass 123456
       }
       virtual_ipaddress {
           192.168.99.152
       }
   }
   virtual_server 192.168.99.151 6701 {
       delay_loop 3
       lb_algo rr
       lb_kind NAT
       persistence_timeout 50
       protocol TCP
       real_server 192.168.99.104 6602 {
           weight 1
       }
   }
   ```

   ​

## CentOS安装Docker

```
yum update -y
yum install -y docker
```

## 常用管理命令

启动、关闭、重启

```
service docker start
service docker stop
service docker restart
```

![](https://images.notes.xuepincat.com/docker/2.png)

左上角的 `DockerFile` 文件定义了镜像要安装程序和配置的环境，通过 `build` 指令可以创建出我们想要的镜像；

一旦创建出镜像，如果想要将镜像分发给其他主机的Docker虚拟机，一种方法是借助Docker仓库来实现，我们可以通过 `push` 指令把本地镜像上传到仓库中，其他主机就可以通过 `search` 指令到仓库中去查找上传的镜像，找到上传的镜像之后可以通过 `pull` 指令将镜像下载到本地，这样别的主机的Docker虚拟机就可以拥有这个镜像了；另一种方式是通过文件的方式，将镜像导出为压缩文件，别的主机再用压缩文件导入为镜像就可以了，导出指令是 `save` 或 `export`，导入的指令是 `load` 或 `import`;

镜像一旦创建出来也是可以删除的，通过 `rmi` 指令可以将镜像删除；

如果想要查看镜像的详细信息，可以使用 `inspect` 指令；

如果想要查看Docker虚拟机内的所有镜像，可以使用 `images` 指令；

镜像是用来创建容器的，从镜像创建出容器的指令是 `run`；

创建出容器之后，这个容器就直接运行了，如果想要停止容器运行或者删除容器，可以使用指令 `pause` 指令暂停，如果恢复运行可以使用 `unpause` 指令；如果想要彻底停止容器的指令是 `stop` ,恢复运行指令为 `start`;

查看容器详细信息可以使用指令 `inspect`;

如果想要查看Docker虚拟机内的所有所有容器可以使用 `ps` 指令，如果删除容器可以使用 `rm` 指令；

容器可以保存为镜像，在容器里面安装程序，配置环境，然后保存为镜像，可以使用 `commit` 指令。

### 安装Java镜像

```
docker search java # 搜索与java相关的的镜像
docker pull java # 下载指定的镜像
```

国外镜像仓库下载速度较慢，建议使用国内镜像仓库，如 `DaoCloud`， [<span style="color:#FF1493;">DaoCloud镜像配置</span>](https://www.daocloud.io/mirror) ， 找到Linux的配置，将其复制粘贴到终端

* Linux下配置

```
curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://f1361db2.m.daocloud.io
```

配置完后还需要修改配置文件

```
vim /etc/docker/daemon.json
```

修改前：

```
{"registry-mirrors": ["http://f1361db2.m.daocloud.io"],}
```

将最后的 `,` 逗号去掉就行了

修改后：

```
{"registry-mirrors": ["http://f1361db2.m.daocloud.io"]}
```

接下来搜索与java有关的镜像

```
docker search java
```

![](https://images.notes.xuepincat.com/docker/3.png)

这里使用的是 `docker.io/java` ，将镜像的名称复制粘贴到下面代码中

![](https://images.notes.xuepincat.com/docker/4.png)

```
docker pull docker.io/java
```

![](https://images.notes.xuepincat.com/docker/5.png)

然后使用下面命令显示docker里面安装的镜像是什么

```
docker images
```

![](https://images.notes.xuepincat.com/docker/6.png)

### 导出导入镜像

安装了Docker镜像，如果想备份镜像把镜像导出，保存为压缩文件，用到的指令是 `save`。如果要从压缩文件导入镜像，使用的指令是 `load`。

#### 语法

```
docker save java > /home/java.tar.gz #导出镜像 
docker load < /home/java.tar.gz #导入镜像
docker images #查看docker虚拟机里面导入导出的镜像有哪些
docker rmi java #删除镜像
```

#### 实操

* 导出刚才安装的java镜像 

![](https://images.notes.xuepincat.com/docker/7.png)

* 查看一下是否导出成功

![](https://images.notes.xuepincat.com/docker/8.png)

* 现在将docker虚拟机里面的镜像删掉

![](https://images.notes.xuepincat.com/docker/9.png)

* 查看镜像

![](https://images.notes.xuepincat.com/docker/10.png)

* 导入镜像

![](https://images.notes.xuepincat.com/docker/11.png)

* 查看镜像

![](https://images.notes.xuepincat.com/docker/12.png)

### 启动容器

#### 语法

```
docker run ...
```

#### 示例

```
docker run -it --name myjava java bash
```

* -it: 启动容器之后开启一个交互的界面
* --name: 给容器起一个名字，可选参数，如果不给容器起名字，它就没有名字，我们管理容器的时候可以根据容器的id去管理容器，比如关闭容器，查看容器信息都可以使用id查找到这个容器
* myjava: 容器的名字
* java: 镜像的名字
* bash:启动的容器运行什么样的程序，运行的是bash命令行

另外还有一些其他参数，比如启动容器之后开启什么端口，把这个端口映射到宿主机上等

```
docker run -it --name myjava -p 9000:8080 -p 9001:8085 java bash
```

* -p: 映射端口
* 9000:8080: `9000` 代表的是宿主机的端口，`:8080` 是容器的端口，这句话的意思说把容器 `8080` 的端口映射到真实主机 `9000` 端口上；
* -p: 映射另外一组端口，容器想映射多少个端口就写多少个 `-p` 参数就可以了；后面的表示把 `8085` 端口映射到真实宿主机`9001` 端口上;

还可以把宿主机上的文件或文件夹映射到容器中,比如将来跑数据库的时候数据库存储的数据一定要保存到宿主机上的，不应该存储到容器里面，数据一定要在容器之外去保存，将来在备份和恢复的时候就非常方便。

```
docker run -it --name myjava -v /home/project:/soft --privileged java bash
```

* -v: 映射文件，如果有多个映射就使用多个 `-v`;
* /home/project:/soft: 宿主机信息，以冒号 `:` 分隔，`/home/project` 表示宿主机的目录，这句表示把宿主机的 `/home/project` 目录映射到容器中的 `/soft` 目录里面;
* --privileged: 在linux系统创建文件和读取文件都是有读写权限的，我把宿主机的目录映射到容器的目录中，`soft` 目录就可以看到真实主机的 `project` 目录中的一些东西了，如果我们想在 `soft` 目录中去创建文件和读写文件，真实的宿主机会不会给 `soft` 这样的权限呢? 后面就需要加上 `--privilged` 这样的参数，这个参数就告诉docker在运行的时候容器在操作映射目录和映射文件的时候是拥有最高权限的，读写都是可以的。

#### 实操

我们首先在 `/home` 目录中创建一个文件夹,将来把文件夹映射到容器里面。

![](https://images.notes.xuepincat.com/docker/13.png)

启动镜像，并将8080端口映射到真实主机9000上，把8085端口映射到9001端口上,还有目录的映射

```
docker run -it -p 9000:8080 -p 9001:8085 -v /home/project:/soft --privileged --name myjava docker.io/java bash
```

![](https://images.notes.xuepincat.com/docker/15.png)

回车后会发现前面的提示都变了，现在的界面是进入到了容器里面，刚才命令我们添加了 `-it` 参数，该参数就是启动一个交互的界面，这里启动的是一个java的容器，里面安装了jdk

* 检测一下java环境，输入 `javac`

![](https://images.notes.xuepincat.com/docker/16.png)

* 查看一下java版本 `java -version`

![](http://images.notes.xuepincat.com/docker/17.png)

* 查看一下映射的目录，会发现没有任何的东西。

![](https://images.notes.xuepincat.com/docker/18.png)

* 在 `soft` 目录中创建一个文件，并向里面写入内容

![](https://images.notes.xuepincat.com/docker/19.png)

* 退出当前容器

![](https://images.notes.xuepincat.com/docker/20.png)

* 进入宿主机的 `/home` 目录查看里面内容

![](https://images.notes.xuepincat.com/docker/21.png)


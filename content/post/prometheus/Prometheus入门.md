+++
title = 'Prometheus入门'
date = 2024-03-04T08:34:19+08:00
draft = false
+++


## 下载并启动

#### 二进制方式启动

```
cd /opt
wget https://github.com/prometheus/prometheus/releases/download/v2.41.0/prometheus-2.41.0.linux-amd64.tar.gz
sudo tar zxvf prometheus-2.41.0.linux-amd64.tar.gz
sudo ln -s /opt/prometheus-2.41.0.linux-amd64 /opt/prometheus
cd /opt/prometheus
sudo ./prometheus
```

![](/img/prometheus/10.png)

当看到 “Server is ready to receive web requests.” 表示启动成功。默认端口9090，可以在浏览器访问 9090 端口:

![](/img/prometheus/20.png)

注意：

prometheus 的 UI 是没有任何用户认证的，如果要想对外暴露，最好做一个nginx代理，然后再做个简单的用户密码认证。

上面是通过执行二进制程序方式启动，如果要停止服务可以【Ctl+C】取消。


#### 开放端口

启动 prometheus 后如果不能通过浏览器立即打开，需要检查防火墙设置：

```
firewall-cmd --zone=public --add-port=9090/tcp --permanent
firewall-cmd --reload
```

#### systemd 方式启动

1、编写 prometheus.service 启动文件

```
[Unit]
Description=Prometheus
Documention=
After=network.target

[Service]
WorkingDirectory=/opt/prometheus/
ExecStart=/opt/prometheus/prometheus
ExecReload=/bin/kill -HUP $MAINPID
ExecStop=/bin/kill -KILL $MAINPID
Type=simple
KillMode=control-group
Restart=on-failure
RestartSec=3s

[Install]
WantedBy=multi-user.target
```

2、将 prometheus.service 文件将文件放入到 /usr/lib/systemd/system 目录下

3、启动

```
sudo systemctl enable prometheus
sudo systemctl status prometheus
sudo systemctl start prometheus
```

![](/img/prometheus/30.png)

## 使用

prometheus 安装好后会默认监控自己，可以点击 Targets 进去查看监控的对象：

![](/img/prometheus/40.png)

点击进去可以查看 prometheus 自己的一些监控指标：

![](/img/prometheus/50.png)

## node_exporter

#### 安装并启动

```
cd /opt
wget https://github.com/prometheus/node_exporter/releases/download/v1.5.0/node_exporter-1.5.0.linux-amd64.tar.gz
sudo tar zxvf node_exporter-1.5.0.linux-amd64.tar.gz
sudo ln -s node_exporter-1.5.0.linux-amd64 /opt/node_exporter
# 启动
cd /opt/node_exporter
sudo ./node_exporter
```

启动后默认端口9100，浏览器访问 9100 端口：

![](/img/prometheus/60.png)

开放端口：

```
firewall-cmd --zone=public --add-port=9100/tcp --permanent
firewall-cmd --reload
```

![](/img/prometheus/70.png)

#### 配置用户认证

所有机器安装完node_exporter后在网络允许的情况下都是可以直接访问的，没有认证，并不安全，所以node_exporter 提供了简单的用户名和密码认证，可以通过 --web.config.file 来配置

具体配置过程参考：https://github.com/prometheus/exporter-toolkit/blob/v0.1.0/https/README.md

1、解压包下默认没有 web.config.yaml 文件，需要手动创建

```
cd /opt/node_exporter
sudo touch web.config.yaml
```

2、配置密码是需要使用 htpasswd，如果没有安装可以通过 httpd-tools 安装：

```
sudo yum install -y httpd-tools
```

3、设置密码

```
$ htpasswd -nB "node"
New password:
Re-type new password:
node:$2y$05$XNqAYRT37Le1vZAaDDCDGOcdz4pwlZYIdea6.al03BX2YqeFRzas2
```

加密，-nB后面的是用户名, 这里我设置的是 node
回车后提示输入密码和确认密码，我设置的密码是 node
最后标准输出用户名:密码：node:$2y$05$XNqAYRT37Le1vZAaDDCDGOcdz4pwlZYIdea6.al03BX2YqeFRzas2


4.配置 web.config.yaml
密码为上一步中得到的密码

```
basic_auth_users:
	node: $2y$05$XNqAYRT37Le1vZAaDDCDGOcdz4pwlZYIdea6.al03BX2YqeFRzas2
```

5、重新启动 node_exporter

```
sudo ./node_exporter --web.config.file="web.config.yaml"
```

启动后重新访问 9100 端口，就会出现下面页面，让你输入用户名和密码：

![](/img/prometheus/80.png)

6、使用 systemd 方式启动 node_exporter，编写 node_exporter.service 启动文件

```
[Unit]
Description=Node Exporter
Documention=
After=network.target

[Service]
WorkingDirectory=/opt/node_exporter/
ExecStart=/opt/node_exporter/node_exporter --web.config.file=/opt/node_exporter/web.config.yaml
ExecStop=/bin/kill -KILL $MAINPID
Type=simple
KillMode=control-group
Restart=on-failure
RestartSec=3s

[Install]
WantedBy=multi-user.target
```

7、和 prometheus 一样，将 node_exporter.service 文件放到 /usr/lib/systemd/system 下，然后启动

```
sudo systemctl enable node_exporter
sudo systemctl status node_exporter
sudo systemctl start node_exporter
```

然后重新访问9100端口测试是否成功启动。

#### 添加 target

![](/img/prometheus/90.png)

现在需要将 node_exporter 加入进来，就需要配置 prometheus.yaml 文件

1、在prometheus.yaml最后面添加上 node 的配置，见最后job_name为node的配置：

```
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]
  - job_name: "node"
    basic_auth:
      username: "node"
      password: "node"
    static_configs:
      - targets: 
      	- "localhost:9100"
```

一般修改完 prometheus.yml 的配置后可以使用自带的 promtool 工具进行检查，比如检查config信息：

```
$ sudo ./promtool check config prometheus.yml
Checking prometheus.yml
 SUCCESS: prometheus.yml is valid prometheus config file syntax

[vagrant@centos-learning prometheus]$
```

配置完后 prometheus 不会主动加载，需要我们自己重新加载：

```
sudo systemctl restart prometheus
```

然后重新刷新下 prometheus 的 target，会发现已经多了 node 端口9100的监控:

![](/img/prometheus/100.png)

但是发现出现了 Error 错误，错误信息为 server returned HTTP status 401 Unauthorized，这是因为之前配置了 用户名和密码的认证,所以还需要在 prometheus.yml 中配置node 的用户名和密码，然后在重新加载 prometheus，添加配置如下：

```
...
- job_name: "node"
    basic_auth:
      username: "node"
      password: "node"
    static_configs:
      - targets: 
      	- "localhost:9100"
```

![](/img/prometheus/110.png)

重启 prometheus 的第二种方式：上面是通过 systemctl restart 的方式重启 prometheus 其实 prometheus 还坚挺了 HUP 信号，如果监听到变化可以重新加载配置文件，可以通过 systemctl staytus 查看下 prometheus 的进程号，然后使用下面命令进行 kill:

```
sudo kill -HUP 6828
```

重启 prometheus 的第三种方式：通过在 prometheus.service 中配置下面参数，然后通过 sudo systemctl reload 重载：

```
...
ExecReload=/bin/kill -HUP $MAINPID
```

systemctl reload实际上是调用 kill -KILL $MAINPID 方式来达到重启目的的：

```
sudo systemctl daemon-reload
或者 
sudo systemctl reload prometheus
```

可以通过 up 查看有哪些节点提供服务：

![](/img/prometheus/120.png)


#### 通过不同端口配置不同机器

1、可以设置不同的端口来模拟不同的机器，比如再添加一台 node_exporter:
node_exporter_02.service:

```
[Unit]
Description=Node Exporter
Documention=
After=network.target

[Service]
WorkingDirectory=/opt/node_exporter/
ExecStart=/opt/node_exporter/node_exporter --web.config.file=/opt/node_exporter/web.config.yaml --web.listen-address=:9101
ExecStop=/bin/kill -KILL $MAINPID
Type=simple
KillMode=control-group
Restart=on-failure
RestartSec=3s

[Install]
WantedBy=multi-user.target
```

2、启动

```
sudo systemctl enable node_exporter_02
sudo systemctl start node_exporter_02
```

3、配置 prometheus.yml

```
...

    static_configs:
      - targets: ["localhost:9090"]
  - job_name: "node"
    basic_auth:
      username: "node"
      password: "node"
    static_configs:
      - targets: 
      	- "localhost:9100"
      	- "localhost:9101"
```

4、重启 prometheus，在打开 prometheus target 页面，就出现了两个 node_exporter target
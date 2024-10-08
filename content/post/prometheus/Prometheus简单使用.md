+++
title = 'Prometheus简单使用'
date = 2024-03-04T08:34:19+08:00
draft = true
categories = [ "Prometheus" ]
+++


## 下载、安装

### 下载

```bash
cd /opt
wget https://github.com/prometheus/prometheus/releases/download/v2.41.0/prometheus-2.41.0.linux-amd64.tar.gz
sudo tar zxvf prometheus-2.41.0.linux-amd64.tar.gz
sudo ln -s /opt/prometheus-2.41.0.linux-amd64 /opt/prometheus
```

### 启动

这里记录两种启动方式。

#### 二进制方式

```bash
cd /opt/prometheus
sudo ./prometheus
```

![](/img/prometheus/10.png)

如上图所示则表示启动成功。默认端口为 `9090`，访问页面如下:

![](/img/prometheus/20.png)

这是通过执行二进制程序方式启动，如果要停止服务可以按【Ctl+C】取消。

启动 prometheus 后如果不能通过浏览器打开，需要检查防火墙设置：

```bash
firewall-cmd --zone=public --add-port=9090/tcp --permanent
firewall-cmd --reload
```

> 注意：<br>
Prometheus 的 UI 界面是没有任何用户认证的，如果要想对外暴露，最好做一个nginx代理，然后再做个简单的用户密码认证。


#### systemd 方式

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

4、验证

prometheus 安装好后会默认监控自己，可以点击【Targets】查看监控的对象：

![](/img/prometheus/40.png)

下面是 prometheus 自己的一些监控指标：

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

1、在prometheus.yaml 最后面 scrape_configs 中添加上 node 的配置，见最后job_name为 node 的配置，scrape_configs 是用来配置监听采集对象的，比如这里配置的 node_exporter：

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

```

配置完后 prometheus 不会主动加载，需要我们自己重新加载：

```
sudo systemctl restart prometheus
```

然后重新刷新下 prometheus 的 target，会发现已经多了 node 端口9100的监控:

![](/img/prometheus/100.png)

但是发现出现了 Error 错误，错误信息为 server returned HTTP status 401 Unauthorized，这是因为之前配置了用户名和密码的认证，所以还需要在 prometheus.yml 中配置node 的用户名和密码，然后在重新加载 prometheus，添加配置如下：

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

重启 prometheus 的第二种方式：上面是通过 systemctl restart 的方式重启 prometheus 其实 prometheus 还监听了 HUP 信号，如果监听到变化可以重新加载配置文件，可以通过 systemctl staytus 查看下 prometheus 的进程号，然后使用下面命令进行 kill:

```
sudo kill -HUP 6828
```

重启 prometheus 的第三种方式：通过在 prometheus.service 中配置下面参数，然后通过 sudo systemctl reload 重载：

```
...
ExecReload=/bin/kill -HUP $MAINPID
```

systemctl reload 实际上是调用 kill -KILL $MAINPID 方式来达到重启目的的：

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

## alertmanager

#### 下载并安装

1、下载

```
cd /opt
wget https://github.com/prometheus/alertmanager/releases/download/v0.25.0/alertmanager-0.25.0.linux-amd64.tar.gz
sudo tar zxvf alertmanager-0.25.0.linux-amd64.tar.gz
sudo ln -s alertmanager-0.25.0.linux-amd64 /opt/alertmanager
```

2、编写 alertmanager.service 启动文件并将 alertmanager.service 文件将文件放入到 /usr/lib/systemd/system 目录下：

```
[Unit]
Description=alertmanager
Documention=
After=network.target

[Service]
WorkingDirectory=/opt/alertmanager/
ExecStart=/opt/alertmanager/alertmanager
ExecReload=/bin/kill -HUP $MAINPID
ExecStop=/bin/kill -KILL $MAINPID
Type=simple
KillMode=control-group
Restart=on-failure
RestartSec=3s

[Install]
WantedBy=multi-user.target
```

3、启动

```
sudo systemctl enable alertmanager
sudo systemctl status alertmanager
sudo systemctl start alertmanager
sudo systemctl status alertmanager
```

4、开放端口

```
firewall-cmd --zone=public --add-port=9093/tcp --permanent
firewall-cmd --reload
```

5、默认端口是9093，浏览器访问9093端口

![](/img/prometheus/130.png)

alertmanager 也是默认没有用户认证的

接下来是如何产生告警，还有产生告警如何发送到 alertmanager?

#### 产生告警

1、告警是在 prometheus 产生，在 prometheus.yml 配置文件中的 rule_files 就是产生告警的配置，可以在 rules_files 下配置 rules/*.yml，表示会匹配 rules 目录下的所有 yml 文件

```
...

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
	- "rules/*.yml"
  # - "first_rules.yml"
  # - "second_rules.yml"

...
```

2、接着在 /opt/prometheus 目录下创建 rules 的文件夹，在 rules 目录下添加 target.yml 文件，不一定名字是 target

```
groups:
	- name: targetdown
	  rules:
  	  - alert: target is down
  	    expr: up == 0
  	    for: 1m
```

定义了一个target 的告警指标，up表示是否在线，1表示在线，0表示离线，查看它的状态，如果是0则进行告警
groups: 表示按类型划分，可以配置多个，所以使用数组
name: 表示名称
rules: 表示规则
alert: 表示告警名称
expr： 查询语句，比如离线就是 up == 0，表示离线则告警
for: 如果出现一次告警则可以不需要配置 for, 如果是在指定分钟内出现告警可以配置 for, 比如1分钟内可以配置 1m

3、可以使用promethes 提供的 tool 也可以检查rule

```
$ sudo ./promtool check rules rules/target.yml
Checking rules/target.ym
  SUCCESS: 1 rules found

[vagrant@centos-learning prometheus]$
```

4、重新加载 prometheus

```
sudo systemctl reload prometheus
```

5、目前node有两台机器，通过不同的端口9100和9101区分

![](/img/prometheus/140.png)

然后停止 node_export 测试

```
sudo systemctl stop node_exporter
```

![](/img/prometheus/150.png)

6、切换 【Alerts】一栏，可以查看到产生一条告警

![](/img/prometheus/160.png)

但是并没有触发告警，告警存在3个状态
因为设置1m，如果在1m中发现了是 up==0 就是 pending 状态，如果超过1m，就是 firing 状态，就会触发告警：

![](/img/prometheus/170.png)

7、现在再重新启动 node_exporter

![](/img/prometheus/180.png)

在状态一栏中rules，通过它进行配置：

![](/img/prometheus/190.png)

#### 告警发送到 alertmanager

现在触发了告警，如何发送到 alertmanager?
1、要让告警能够发送到 alertmanager ，就需要让它们进行关联，需要配置 prometheus.yml 的 alertmanager ，然后重载 prometheus

```
...

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - "localhost:9093"
          # - alertmanager:9093

...
```

2、可以在 build 一栏中看到下面信息则表示 alertmanager 已配置

![](/img/prometheus/200.png)

3、再次停止 node_exporter 使 up ==0 离线产生告警

```
sudo systemctl stop node_exporter
```

![](/img/prometheus/210.png)

等待1m触发告警：

![](/img/prometheus/220.png)

4、打开 alertmanager，可以看到产生一条告警

![](/img/prometheus/230.png)

5、可以点击 Source 进行跳转

![](/img/prometheus/240.png)

如果发现跳转页面打不开：可以替换 cenots-learning 为你的 IP主机IP地址：

![](/img/prometheus/250.png)

![](/img/prometheus/260.png)

实际上跳转的就是对应的告警规则查询页面。
至于跳转的 url 名为是 centos-learning 是因为我的主机设置就是，在 prometheus 提供了 web.external-url 来自行指定IP

![](/img/prometheus/270.png)

6、也可以设置静默，可以自行根据需要设置静默时间：

![](/img/prometheus/280.png)

![](/img/prometheus/290.png)

7、.接着启动 node_exporter 恢复

```
sudo systemctl start node_exporter
```

恢复后在 alertmanager 查看告警名称会发现已经没有记录了，表示告警已恢复

![](/img/prometheus/300.png)

prometheus UI相对比较简单，可以通过 grafana 来展示

## 基本概念

### 数据模型

Prometheus 将所有数据存储在时序数据库中，每个时间序列由指标名称和标签键值对唯一确定

格式：<metric name>{<label name>=<label value>, ...}  samples [millisecond]

指标名称: [a-zA-A_:][a-zA-Z0-9_:]*

标签名：[a-zA-A_][a-zA-Z0-9_]*

指标值：任意格式的Unicode值

采样数据： float64格式

### 指标类型
prometheus client 库提供4种核心度量类型，prometheus server 并不使用类型信息
Counter: 
计数类
使用在累计指标单调递增或单调递减情况下，只能在目标重启后自动归零
比如：服务器请求处理数量，已完成任务数量，错误数量

Guage
测量类
可大可小的
使用可增可减数据情况下
比如：当前内存使用情况、并发请求数量

Histogram
直方图类
使用统计指标信息在不同区间内（桶）的统计数量
比如：5分钟内的响应数据量多少，延迟时间，响应大小

Summary  
摘要类
类似直方图，在客户端对百分位进行统计
比如：延迟时间，响应大小

job
具有相同目的的实例集合

instance
采集的端点，目标

## 配置

### 命令行参数

启动后不可修改，除非重启进程

### 配置文件

* 通过命令行 --config.file 指定 
* 可修改并通过信号或 API 重新加载

1、通过信号，如：kill -s SIGHUP $PID
2、通过 API，通过 API 需要启动 /-/reload，然后再通过命令行参数 ---web.enable-lifecycle 开启
参考：https://prometheus.io/docs/prometheus/latest/configuration/configuration/

* 全局配置

#### 配置 query_log_file

1.修改前目录下没有日志：

![](/img/prometheus/310.png)

2、修改 prometheus.yml

```
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  query_log_file: "prometheus.log"
  # scrape_timeout is set to the global default (10s).
...
```

3、重启后查看日志

![](/img/prometheus/320.png)

内容如下：

```
[root@localhost prometheus]# cat prometheus.log
{"params":{"end":"2023-12-22T00:13:11.536Z","query":"up == 0","start":"2023-12-22T00:13:11.536Z","step":0},"ruleGroup":{"file":"rules/target.yml","name":"targetdown"},"spanID":"0000000000000000","stats":{"timings":{"evalTotalTime":0.004941969,"resultSortTime":0,"queryPreparationTime":0.002010032,"innerEvalTime":0.002638045,"execQueueTime":0.001544889,"execTotalTime":0.006733258},"samples":{"totalQueryableSamples":3,"peakSamples":8}},"ts":"2023-12-22T00:13:11.552Z"}
{"params":{"end":"2023-12-22T00:13:26.536Z","query":"up == 0","start":"2023-12-22T00:13:26.536Z","step":0},"ruleGroup":{"file":"rules/target.yml","name":"targetdown"},"spanID":"0000000000000000","stats":{"timings":{"evalTotalTime":0.007906033,"resultSortTime":0,"queryPreparationTime":0.003400991,"innerEvalTime":0.00415388,"execQueueTime":0.001132826,"execTotalTime":0.009505957},"samples":{"totalQueryableSamples":3,"peakSamples":8}},"ts":"2023-12-22T00:13:26.556Z"}

...
{"httpRequest":{"clientIP":"192.168.64.1","method":"GET","path":"/api/v1/query"},"params":{"end":"2023-12-22T00:14:43.615Z","query":"up","start":"2023-12-22T00:14:43.615Z","step":0},"spanID":"0000000000000000","stats":{"timings":{"evalTotalTime":0.003373893,"resultSortTime":0,"queryPreparationTime":0.002024,"innerEvalTime":0.001140089,"execQueueTime":0.000155886,"execTotalTime":0.003687061},"samples":{"totalQueryableSamples":3,"peakSamples":3}},"ts":"2023-12-22T00:14:43.554Z"}
```

#### 外部标签 external_label

作用：
prometheus 往外发的数据中加的 label，需要注意的这只是对外发送，而不是内部使用的
比如对外发送 alertmanager 的时候会带上这些标签

场景：
比如有多个 prometheus server，把数据发送到一个 alertmanager 上的时候，就可以通过 external_label 来区分是哪个 prometheus 发送的

1、定义 external_labels

```yaml
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  query_log_file: "prometheus.log"
  external_labels:
    env: test
  # scrape_timeout is set to the global default (10s).
...
```

2、重启 prometheus

```
systemctl reload prometheus
```

3、先暂停掉 node_exporter

```
systemctl stop node_exporter
```

## scrape_config

job_name 必须要配置

scrape_interval: 可以对每个 job 设置单独的爬取时间

scrape_timeout: 也可以定义爬取的超时时间

metrics_path: 每个采集目标url可以是不一样的，默认 prometheus 请求的是 /metrics
 
honor_label: 
prometheus 默认会携带一些标签的，比如 instance、job，但这些 label 可能和采集指标时名称冲突，此时就需要决定谁覆盖谁，或者重命名，
这种情况，prometheus 会将爬取过来的采集目标的指标标签改为 exported_instance
有时候不能重命名，需要保留，比如 pushgateway 和 prometheus 级联的时候需要保留原来爬取过来的

honor_label 默认是false，表示会重命名


抓取有介绍两个配置一个是 static config，一个就是 file_sd_config 配置

### static_config

略

### file_sd_config 服务发现

需要配置两个东西，一个是 files，一个是刷新的时间，即每隔多久刷新这些文件，当然当文件发生变化，prometheus 也会重新加载文件进行服务发现。

1、编辑 prometheus.yml:

```
...

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
      username: node
      password: node
    # static_configs:
    #   - targets:
    #     - "localhost:9100"
    #     - "localhost:9101" 
  	file_sd_configs:
  	  - files:
    	  - sd/files/node/*.yaml
```

2、重载 prometheus ，并检查check :

```
sudo systemctl reload prometheus
./promtool check config prometheus.yml
```

3、检查配置时提示下面信息,表示 sd/files/node/*.yaml 目录并不存在：

```
Checking prometheus.yml
  WARNING: file "sd/files/node/*.yaml" for file_sd in scrape job "node" does not exist
  SUCCESS: 1 rule files found
 SUCCESS: prometheus.yml is valid prometheus config file syntax

Checking rules/target.yml
  SUCCESS: 1 rules found

```

4、可以到 prometheus 目录下新建 sd/files/node/default.yaml 文件，然后在重新check config，文件名任取：

```
$ pwd
/opt/prometheus
$ sudo mkdir -p sd/files/node
$ sudo touch sd/files/node/default.yaml
$ ./promtool check config prometheus.yml
Checking prometheus.yml
  SUCCESS: 1 rule files found
 SUCCESS: prometheus.yml is valid prometheus config file syntax

Checking rules/target.yml
  SUCCESS: 1 rules found

```

5、然后重新加载 promethes，在查看prometheus config

```
systemctl reload prometheus
```

![](/img/prometheus/330.png)

node 也消失了：

![](/img/prometheus/340.png)

6、接下来配置defautl.yaml 文件

```
- targets: ["localhost:9100"]
```

配置完保存打开target页面会发现自动加载：

![](/img/prometheus/350.png)
![](/img/prometheus/360.png)

服务发现回去发现本地文件变化后会重新加载，也就是服务发现中配置的文件变化后是不需要手动reload 的，它会自动重新加载，但是是 prometheus 本身的文件或者配置的其他比如 rules 的文件变化还是需要我们手动去重载的

如果再进行配置 defautl.yaml：

```
- targets: ["localhost:9100", "localhost:9101"]
```

也会自动加载:

![](/img/prometheus/370.png)

接着将 9100 的配置删掉：

```
- targets: ["localhost:9101"]
```

![](/img/prometheus/380.png)

除了配置yaml 也可以配置json，编辑 prometheus.yml，编辑完后重载 prometheus，并查看 configuation 查看是够将 json 配置添加进来:

```
...

    static_configs:
      - targets: ["localhost:9090"]
  - job_name: "node"
    basic_auth:
      username: node
      password: node
    # static_configs:
    #   - targets:
    #     - "localhost:9100"
    #     - "localhost:9101"
  	file_sd_configs:
  	  - files:
    	  - sd/files/node/*.yaml
    	  - sd/files/node/*.json
```

添加个 default.json 文件，新增文件也可以被监听到，因为上面将 9100 删掉了，所以这里配置的 9100：


```
[
    {
        "targets": [
            "localhost:9100"
        ]
    }
]
```

![](/img/prometheus/390.png)

有个很奇怪的现象，如果配置个不存在的 比如 9102,也是可以抓取到的，但是状态会异常：

```
[
    {
        "targets": [
            "localhost:9100",
            "localhost:9102"
        ]
    }
]
```

![](/img/prometheus/400.png)
![](/img/prometheus/410.png)


7、还原回去，将不存在的抓取目标移除


## Alerting

prometheus 规则有两种：
一种是产生告警，一种是生成新的时间序列 ，因为有些数据并没有给直接统计出来，而是现查，有时候现查比较慢就需要根据周期性数据生成新的时间序列，然后产生的告警基于新的时间序列进行验证，比如 CPU 核数、使用率

### 说明

groups: 配置规则组
name: 规则组名称
interval: 规则运行时间间隔
rules: 规则列表

#### 举例 创建node规则，新生成CPU核数

rules/node.yml:

```
groups:
  - name: node
    rules:
      - record: node_cpu_total
        expr: count(node_cpu_seconds_total{mode="system"})by(instance)
```

上面的 rule 就生成了一个新的时间序列，重新加载后查看 rules:

![](/img/prometheus/420.png)

prometheus.yml 修改全局配置，设置抓取时间为 15s， scrape_interval 和 evaluation_interval:

```
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

...  
```

接着进行查询：

![](/img/prometheus/430.png)

接着再 node.yml 中添加CPU使用率：

```
groups:
  - name: node
    rules:
      - record: node_cpu_total
        expr: count(node_cpu_seconds_total{mode="system"})by(instance)
      - record: node_cpu_percent
        expr: sum(1-irate(node_cpu_seconds_total{mode="idle"}[5m]))by(instance)*100
      - record: node_mem_percent
        expr: (1-node_memory_MemAvailable_bytes/node_memory_MemTotal_bytes)*100
```

重载prometheus后查看下rules:

![](/img/prometheus/440.png)

接着查询写：

![](/img/prometheus/450.png)

使用 top查看下：

![](/img/prometheus/460.png)

### 告警注释

rules/target.yml:

```yaml
groups:
  - name: targetdown
    rules:
      - alert: target is down
        expr: up == 0
        for: 1m
        labels:
          level: heigh
        annotations:
          summary: "节点离线"
          description: "节点离线"
```

给 node 定义规则，rules/node.yml:

```yaml
groups:
  - name: node
    rules:
      - record: node_cpu_total
        expr: count(node_cpu_seconds_total{mode="system"})by(instance)
      - record: node_cpu_percent
        expr: sum(1-irate(node_cpu_seconds_total{mode="idle"}[5m]))by(instance)*100
      - record: node_mem_percent
        expr: (1-node_memory_MemAvailable_bytes/node_memory_MemTotal_bytes)*100
  - name: node_alert
    rules:
      - alert: "CPU 使用率告警"
        expr: node_cpu_percent > 80
        for: 1m
        labels:
          level: heigh
        annotations:
          summary: "CPU 使用率过高"
          description: "CPU 使用率过高"
      - alert: "内存使用率告警"
        expr: node_mem_percent > 60
        for: 1m
        labels:
          level: heigh
        annotations:
          summary: "内存使用率过高"
          description: "内存使用率过高"
```

更新后重载，并查看rules 配置：

![](/img/prometheus/470.png)

手动打高CPU使用率:

```
cat /dev/zero > /dev/null
```

![](/img/prometheus/480.png)

![](/img/prometheus/490.png)

![](/img/prometheus/500.png)

也会看到一条alertmanager 的告警：

![](/img/prometheus/510.png)

可以看到注释只会在 prometheus 中看到，并不会将注释发送到 alertmanager 中。

另外注释用还可以添加一些变量来展示一些节点信息，让用户可以知道哪个节点，修改 node.yml：

```
groups:
  - name: node
    rules:
      - record: node_cpu_total
        expr: count(node_cpu_seconds_total{mode="system"})by(instance)
      - record: node_cpu_percent
        expr: sum(1-irate(node_cpu_seconds_total{mode="idle"}[5m]))by(instance)*100
      - record: node_mem_percent
        expr: (1-node_memory_MemAvailable_bytes/node_memory_MemTotal_bytes)*100
  - name: node_alert
    rules:
      - alert: "CPU 使用率告警"
        expr: node_cpu_percent > 80
        for: 1m
        labels:
          level: heigh
        annotations:
          summary: "节点 {{$labels.instance}} CPU 使用率过高 {{$labels.value}}"
          description: "CPU 使用率过高"
      - alert: "内存使用率告警"
        expr: node_mem_percent > 60
        for: 1m
        labels:
          level: heigh
        annotations:
          summary: "内存使用率过高"
          description: "内存使用率过高"
```

上面的 cpu 中没有job, 因为 by 的时候没有设置 job:

```
groups:
  - name: node
    rules:
      - record: node_cpu_total
        expr: count(node_cpu_seconds_total{mode="system"})by(instance)
      - record: node_cpu_percent
        expr: sum(1-irate(node_cpu_seconds_total{mode="idle"}[5m]))by(instance,job)*100
      - record: node_mem_percent
        expr: (1-node_memory_MemAvailable_bytes/node_memory_MemTotal_bytes)*100
  - name: node_alert
    rules:
      - alert: "CPU 使用率告警"
        expr: node_cpu_percent > 80
        for: 1m
        labels:
          level: heigh
        annotations:
          summary: "节点 {{$labels.job}}.{{$labels.instance}} CPU 使用率过高 {{$value}}"
          description: "CPU 使用率过高"
      - alert: "内存使用率告警"
        expr: node_mem_percent > 60
        for: 1m
        labels:
          level: heigh
        annotations:
          summary: "节点 {{$labels.job}}.{{$labels.instance}} 内存使用率过高 {{$value}}"
          description: "内存使用率过高"
```

![](/img/prometheus/520.png)

## PROMQL

略

## HTTP API 

prometheus 中 API 共有三类：

1、查询类

2、管理员类

3、生命周期管理类

### 管理类

1、通过命令行参数 --web.enable-admin-api 启动

2、快照

路径： /api/v1/admin/tsdb/snapshot
请求方式：POST/PUT
查询参数：skip_head（跳过头数据）

3、删除时序

路径： /api/v1/admin/tsdb/delete_series
请求方式：POST/PUT
查询参数：
match[]、start、end

删除并未真正从硬盘删除，后续在压缩时进行清理

4、清理磁盘数据

路径： /api/v1/admin/tsdb/clean_tombstones
请求方式：POST/PUT  

### 生命周期管理类

1、命令行参数通过 --web.enable-lifecycle 启动

2、健康状态

路径： /-/healthy
请求方式：GET

3、装备状态

路径：/-/ready
请求方式：GET

4、重新加载配置

路径：/-/reload
请求方式：POST

5、退出

路径：/-/quit
请求方式：POST


## Prometheus 开发

1、基于 promtheus 开发应用，需要熟悉 prometheus API， 配置， PromQL 等

如：
服务发现 => CMDB prometheus  Target 配置
图形化管理 => 可视化
告警管理（告警一旦恢复就看不到历史记录了，可以将 alertmanager 发送高 cmdb 告警平台）
多个 prometheus 管理

2、prometheus
如：
补丁
服务发现  


二次开发，需要熟悉哪些事？
1、配置文件 yaml 操作
2、HTTP API -> http client  
  prometheus 增删改查
    id, name, ip
  job 增删改查
    id, prometheus, jobnam, schema, metrics,basic_auth, tls

  target:
    target

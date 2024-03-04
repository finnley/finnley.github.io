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

点击进去可以查看prometheus 自己的一些监控指标：

![](/img/prometheus/50.png)
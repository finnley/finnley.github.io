+++
title = 'Prometheus入门'
date = 2024-03-04T08:34:19+08:00
draft = true
+++


## 下载并启动

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
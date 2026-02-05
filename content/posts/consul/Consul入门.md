+++
title = 'Consul入门'
date = 2026-01-03T09:46:28+08:00
draft = true
categories = [ "Consul" ]
tags = [ "consul"]
+++

## 安装与配置

**1. 安装**

```shell
docker run -d -p 8500:8500 -p 8300:8300 -p 8301:8301 -p 8302:8302 -p 8600:8600/udp  consul consul agent  -dev -client=0.0.0.0

docker container update --restart=always 容器名字
```

8500 是HTTP端口，8600是DNS端口

**2. 访问**

安装完成后通过`8500`端口打开，页面有一个默认consul服务，这是Consul自己注册进来的默认服务。
![Consul可视化预览](/images/consul/10.png)

Consul提供了dns功能，可以让我们通过dig命令行来测试，consul默认的dns端口是8600，dig 可以从指定DNS服务器上将DNS地址解析出IP和端口。

Linux下的dig命令安装：
```shell
yum install -y bind-utils
```

使用dig命令，它可以将指定的DNS服务器上的DNS地址解析出IP和端口：
```shell
dig @192.168.20.21 -p 8600 consul.service.consul SRV
```

完整示例如下：
```shell
[vagrant@centos7-dev ~]$ dig @192.168.20.21 -p 8600 consul.service.consul SRV

; <<>> DiG 9.11.4-P2-RedHat-9.11.4-26.P2.el7_9.3 <<>> @192.168.20.21 -p 8600 consul.service.consul SRV
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 47553
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 3
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;consul.service.consul.		IN	SRV

;; ANSWER SECTION:
consul.service.consul.	0	IN	SRV	1 1 8300 a28be063200a.node.dc1.consul.

;; ADDITIONAL SECTION:
a28be063200a.node.dc1.consul. 0	IN	A	127.0.0.1
a28be063200a.node.dc1.consul. 0	IN	TXT	"consul-network-segment="

;; Query time: 31 msec
;; SERVER: 192.168.20.21#8600(192.168.20.21)
;; WHEN: Sun Feb 28 01:34:35 UTC 2021
;; MSG SIZE  rcvd: 150

[vagrant@centos7-dev ~]$
```

DNS 默认端口是53，Consul为了防止冲突，就提供了 8600 的端口，对于每个服务，比如SRV，都会生成对应的域名，比如 consul.service.consul ，域名后缀是 service.consul ，前面的 consul 是 service name 或者 id，打开的时候已经默认有了一个 consul。



## API接口

### 服务注册

[文档](https://www.consul.io/api-docs/agent/service#register-service)

```shell
/v1/agent/service/register
```

```python
import requests

headers = {
    "contentType": "application/json"
}


def register(name, id, address, port):
    url = "http://127.0.0.1:8500/v1/agent/service/register"
    rsp = requests.put(url, headers=headers, json={
        "Name": name,
        "ID": id,
        "Tags": ["consul", "python", "test"],
        "Address": address,
        "Port": port
    })
    if rsp.status_code == 200:
        print("服务注册成功")
    else:
        print(f"服务注册失败：{rsp.status_code}")


if __name__ == "__main__":
    register("user-service", "xuepingmall-user-service", "127.0.0.1", 50051)
    # deregister("mshop-web")
```

![Consul可视化预览](/images/consul/20.png)
![Consul可视化预览](/images/consul/30.png)

### 服务注销

```python
def deregister(id):
    url = f"http://127.0.0.1:8500/v1/agent/service/deregister/{id}"
    rsp = requests.put(url, headers=headers)
    if rsp.status_code == 200:
        print("服务注销成功")
    else:
        print(f"服务注销失败：{rsp.status_code}")


if __name__ == "__main__":
    deregister("xuepingmall-user-service")
```

### 健康检查

**HTTP**

需要提前准备一个专门用于健康检查的接口，比如：
```code
http://192.168.1.24:8021/health
{
    "code": 200,
    "success": true
}
```

在服务注册时做健康检查，代码如下：
```python
import requests

headers = {
    "contentType": "application/json"
}


# 在注册服务时设置健康检查
def register(name, id, address, port):
    url = "http://127.0.0.1:8500/v1/agent/service/register"
    rsp = requests.put(url, headers=headers, json={
        "Name": name,
        "ID": id,
        "Tags": ["consul", "python", "test"],
        "Address": address,
        "Port": port,
        "Check": {
            "HTTP": f"http://{address}:{port}/health",  # 当consul拿到这个 key 是 HTTP 就会取出自己的http包向指定的url发送请求
            "Timeout": "5s",
            # 5s 超时 最小 timeout 1min,如果服务挂掉了，consul库在请求接口时不会立马感受到挂掉，可能会一直等着返回，这里5s并不是说超时了，而是仍然会采用自己的1min
            "Interval": "5s",  # 检查频率
            "DeregisterCriticalServiceAfter": "15s"  # 如果5s没有检查到，发生严重错误5s之后就注销掉
        }
    })

    if rsp.status_code == 200:
        print("服务注册成功")
    else:
        print(f"服务注册失败: {rsp.status_code}")


if __name__ == "__main__":
    register("user-service-correct", "xuepingmall-user-correct-service", "192.168.1.24", 8021)  # consul 是在虚拟机容器中，gin服务是在本地，对于虚拟机来说本地机器的IP不是127.而是192开头
    register("user-service-incorrect", "xuepingmall-user-incorrect-service", "127.0.0.1", 50051)

```

**gRPC**

任何一个服务注册中心如果想对某一个gRPC服务做健康检查，需要遵循gRPC提供的 [Service Definition](https://github.com/grpc/grpc/blob/master/doc/health-checking.md)，需要我们自己去写Proto然后去调用即可。

服务端的实现 gRPC已经为我们完成了，我们只需要注册下gRPC服务即可。

grpc健康检查重要点:
1. check = {
       "GRPC": f'{ip}:{port}',
       "GRPCUseTLS": False,
       "Timeout": "5s",
       "Interval": "5s",
       "DeregisterCriticalServiceAfter": "5s",
    }
2. 一定要确保网络是通的
3. 一定要确保srv服务监听端口是对外可访问的
4. GRPC一定要自己填写

**第三方库**




### 批量注册

### 服务获取
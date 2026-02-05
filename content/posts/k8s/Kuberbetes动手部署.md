+++
title = 'Kuberbetes动手部署'
date = 2026-01-21T17:18:11+08:00 
draft = true
categories = [ "Kubernetes" ]
tags = [ "k8s", "kubernetes" ]
+++

## 目标

- 在所有节点上安装Docker、kubeadm、kubelet
- 部署容器网络插件flannel

## 架构

|       IP         |      域名     |      备注     |                  安装软件                    |
| :---             |    :----:    |    :----:     |                  ---:                      |
| 172.20.30.1      |     master   |      主节点    | Docker、kubeadm、kubelet、kubectl、flannel   |
| 172.20.30.2      |     node1    |      从节点1   | Docker、kubeadm、kubelet、kubectl、flannel   |
| 172.20.30.3      |     node2    |      从节点2   | Docker、kubeadm、kubelet、kubectl、flannel   |

## 环境准备

- 3台虚拟机CentOS7.x-x86_x64  
- 硬件配置：2GB或更多RAM，2个CPU或更多CPU，硬盘30GB或更多  
- 集群中所有机器之间网络互通  
- 可以访问外网，需要拉取镜像  
- 禁止swap分区

## 安装基础软件

- 配置Master与work节点域名

```shell
vim /etc/hosts
172.20.30.1 master
172.20.30.2 node1
172.20.30.3 node2
```

- 设置域名解析服务器
```shell
vim /etc/resolv.conf
nameserver 114.114.114.114
```

**安装软件**

```shell
yum install -y wget net-tools ntp
```

**同步系统时间**
```shell
ntpdate 0.asia.pool.ntp.org
```

**安装Docker**
略

**配置Docker、Kuberetes镜像源**

```shell
cat <<EOF | tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.28/rpm/
enabled=1
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.28/rpm/repodata/repomd.xml.key
EOF

cat <<EOF | tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.24/rpm/
enabled=1
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.24/rpm/repodata/repomd.xml.key
EOF
setenforce 0
yum install -y kubelet kubeadm kubectl
systemctl enable kubelet && systemctl start kubelet

```

**将桥接的IPV4流量传递到iptables的链**

```shell
modprobe br_netfilter
echo "1" >/proc/sys/net/bridge/bridge-nf-call-iptables
vi /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
```

**关闭防火墙**

```shell
systemctl stop firewalld
systemctl disable firewalld
```

**关闭SeLinux**
```shell
setenforce 0
sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config
```

**关闭swap**

```shell
swapoff -a
yes | cp /etc/fstab /etc/fstab_bak
```

## Master节点安装 kubeadm, kubelet and kubectl

- 首先确保虚拟机的 CPU 为 2 核（Kubeadm init 要求 2 核）
- 修改docker配置文件，使用 systemd 作为 cgroup 的驱动（Kubeadm init 推荐）

```shell
mkdir /etc/docker

cat > /etc/docker/daemon.json <<EOF
{
  "registry-mirrors": ["https://registry.cn-hangzhou.aliyuncs.com"]
}
EOF
```

**安装核心组件**

```shell
yum install -y kubeadm kubelet kubectl
```

## 初始化Master节点

**设置主机名**

```shell
hostnamectl set-hostname master
```

**初始化主节点**

```shell
kubeadm init --kubernetes-version=v1.24.0 \
--apiserver-advertise-address=172.20.30.1 \
--image-repository registry.aliyuncs.com/google_containers \
--service-cidr=10.1.0.0/16 \
--pod-network-cidr=10.244.0.0/16

kubeadm init --ignore-preflight-errors=FileContent--proc-sys-net-bridge-bridge-nf-call-iptables \
             --ignore-preflight-errors=FileContent--proc-sys-net-bridge-bridge-nf-call-ip6tables \
             --ignore-preflight-errors=SystemVerification \
             --apiserver-advertise-address=172.20.30.1 \
--image-repository registry.aliyuncs.com/google_containers \
--service-cidr=10.1.0.0/16 \
--pod-network-cidr=10.244.0.0/16
```

---

cp /etc/containerd/config.toml /etc/containerd/config.toml.bak 2>/dev/null
rm -f /etc/containerd/config.toml
systemctl restart containerd

```
systemctl status containerd
systemctl is-active containerd   # 应该返回 active

# 备份（可选）
cp /etc/containerd/config.toml /etc/containerd/config.toml.bak 2>/dev/null

# 直接删除旧配置，让 containerd 下次启动时用默认配置（默认已启用 CRI）
rm -f /etc/containerd/config.toml

systemctl restart containerd

# 再试一次初始化
kubeadm init ...
```

---


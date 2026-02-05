+++
title = 'K8s集群问题整理'
date = 2024-10-18T21:46:45+08:00
draft = true
categories = [ "Programming" ]
tags = [ "k8s", "kubernetes" ]
+++

## 1 OverlayFS-on-OverlayFS（文件系统嵌套）兼容性问题

### 1.1 背景

K8s In Docker。

宿主机环境为Ubuntu24.04，在宿主机上使用Docker创建了一个也是Ubuntu24.04的容器作为系统，然后在这个容器系统中搭建k8s集群。

### 1.2 问题

```shell
# kind create cluster --name mycluster \
  --image m.daocloud.io/docker.io/kindest/node:v1.35.0
Creating cluster "mycluster" ...
 ✓ Ensuring node image (m.daocloud.io/docker.io/kindest/node:v1.35.0) 🖼 
 ✗ Preparing nodes 📦  
ERROR: failed to create cluster: command "docker run --name mycluster-control-plane --hostname mycluster-control-plane --label io.x-k8s.kind.role=control-plane --privileged --security-opt seccomp=unconfined --security-opt apparmor=unconfined --tmpfs /tmp --tmpfs /run --volume /var --volume /lib/modules:/lib/modules:ro -e KIND_EXPERIMENTAL_CONTAINERD_SNAPSHOTTER --detach --tty --label io.x-k8s.kind.cluster=mycluster --net kind --restart=on-failure:1 --init=false --cgroupns=private --publish=127.0.0.1:32865:6443/TCP -e KUBECONFIG=/etc/kubernetes/admin.conf m.daocloud.io/docker.io/kindest/node:v1.35.0" failed with error: exit status 125
Command Output: docker: Error response from daemon: failed to mount /tmp/containerd-mount4213980840: mount source: "overlay", target: "/tmp/containerd-mount4213980840", fstype: overlay, flags: 0, data: "workdir=/var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/3/work,upperdir=/var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/3/fs,lowerdir=/var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/2/fs:/var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/1/fs,index=off", err: invalid argument
需要我再提供哪些信息给你
```

该问题不仅针对Kind方式创建的集群，也针对kubadm方式搭建的集群。

### 1.3 分析

这是一个典型的 **OverlayFS-on-OverlayFS**（文件系统嵌套）兼容性问题。

简单来说，Kind 试图在 Docker 容器内部再创建一个基于 OverlayFS 的容器（Kind 节点），但因为你的当前环境（外层容器）本身就是运行在 OverlayFS 之上的，而 Linux 内核通常不支持（或需要特殊配置才能支持）这种多层嵌套，导致挂载失败并报 `invalid argument` 错误。

问题根源：Kind (Containerd) 试图在 Docker (OverlayFS) 之上再创建一个 OverlayFS 挂载。Linux 内核通常禁止 OverlayFS over OverlayFS。

### 1.4 解决方案

在 docker-compose.yaml 中为容器添加一个匿名卷（Volume）挂载到 /var/lib/docker。 这样做会让内部 Docker 的数据目录直接落在宿主机的磁盘文件系统（如 ext4 或 xfs）上，而不是落在外层容器的 OverlayFS 层上，从而避开“套娃”冲突。

```yaml
volumes:
  - /var/lib/docker  # <--- 【新增这一行】关键修复！
  # 建议（可选）：为了让 Kind 更好地支持网络功能，建议把宿主机内核模块也挂载进去
  - /lib/modules:/lib/modules:ro
```

然后重启宿主机容器并重新部署。

## 2 preflight错误

### 2.1 问题

**命令**
```shell
kubeadm init --kubernetes-version=v1.35.0 \
    --image-repository registry.aliyuncs.com/google_containers \
    --cri-socket=unix:///var/run/containerd/containerd.sock \
    --apiserver-advertise-address=172.20.30.1 \
    --pod-network-cidr=10.244.0.0/16 \
    --v=5
```

**错误**
```shell
[preflight] Some fatal errors occurred:
        [ERROR SystemVerification]: failed to parse kernel config: unable to load kernel module: "configs", output: "", err: exec: "modprobe": executable file not found in $PATH
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
error: error execution phase preflight: preflight checks failed
```

### 2.2 分析

错误是因为 **`kubeadm` 在做“预检”（Preflight Check）**。它试图运行 `modprobe` 命令来检查内核配置（是否加载了 `configs` 模块），但我的容器基础镜像（Ubuntu Minimal）非常精简，**默认没有安装 `kmod`（包含 modprobe 的包）**。

在容器环境中部署 K8s，既然内核是共享宿主机的，容器内部其实并没有权限（也没必要）去加载内核模块。

我有两个选择来解决这个问题：

方案一：忽略预检错误（推荐 ✅）

这是最简单、最符合容器化场景的做法。既然我无法在容器里改变宿主机内核，检查也没意义，直接跳过它。

请在命令中添加 `--ignore-preflight-errors=SystemVerification` 参数：

```bash
kubeadm init --kubernetes-version=v1.35.0 \
    --image-repository registry.aliyuncs.com/google_containers \
    --cri-socket=unix:///var/run/containerd/containerd.sock \
    --apiserver-advertise-address=172.20.30.1 \
    --pod-network-cidr=10.244.0.0/16 \
    --v=5 \
    --ignore-preflight-errors=SystemVerification

```

*(注：如果有其他预检错误，比如 Swap 相关的，你可以用逗号分隔，如 `--ignore-preflight-errors=SystemVerification,Swap`)*

方案二：安装缺失的工具（仅作补充）

如果你非要通过检查（虽然可能还会因为找不到内核模块文件而报错），可以安装该工具：

```bash
apt-get update && apt-get install -y kmod

```

*安装后再次运行原命令即可。但通常安装后它会报下一个错（如找不到 `/lib/modules`），所以还是推荐方案一。*

---


## 3 Swap未关闭

禁用的原因：当内存不够时开启Swap会使用磁盘来代替内存，由于内存和磁盘不是同一计算等级，禁用掉就不会出现计算速率不一致的问题。

编辑`/etc/fstab`文件，注释`/swapfile ...`或`/swap.img`这一行，然后执行`swapoff -a`重启。如果是`K8s in Docker`模式，需要禁用宿主机的Swap分区。

## 4 CRI 插件未启用

指定了 Docker 自带的 `containerd.sock`。默认情况下，Docker 安装的 containerd **禁用了 CRI 插件**（`disabled_plugins = ["cri"]`）。如果不修改 `/etc/containerd/config.toml`，kubeadm 虽然能初始化成功，但 **kubelet 永远无法启动**，导致集群一直卡在等待状态。


### ⚠️ 高能预警：下一步可能会遇到的坑

解决这个问题后，`kubeadm init` 可能会卡在或者报错以下两个常见问题，建议提前检查：

1. **Swap未关闭**：K8s 要求关闭 Swap。如果报错 `[ERROR Swap]`，请在上面的 ignore 参数中加上 `,Swap`。
2. **CRI 插件未启用**：你指定了 Docker 自带的 `containerd.sock`。默认情况下，Docker 安装的 containerd **禁用了 CRI 插件**（`disabled_plugins = ["cri"]`）。如果不修改 `/etc/containerd/config.toml`，kubeadm 虽然能初始化成功，但 **kubelet 永远无法启动**，导致集群一直卡在等待状态。

**建议你现在就检查一下 containerd 配置：**

```bash
# 查看配置文件是否禁用 cri
grep "disabled_plugins" /etc/containerd/config.toml

```

如果看到 `cri` 被禁用了，执行以下修复命令后再运行 `kubeadm init`：

```bash
# 生成默认配置并覆盖（开启 cri）
containerd config default > /etc/containerd/config.toml
# 修改沙箱镜像地址为阿里云（解决国内网络问题）
sed -i 's/registry.k8s.io\/pause:3.6/registry.aliyuncs.com\/google_containers\/pause:3.9/g' /etc/containerd/config.toml
# 重启 containerd
systemctl restart containerd

```

## 解决kube-proxy崩溃 (CrashLoopBackOff)问题

原因：在 Docker 容器（DinD）中运行 K8s 时，kube-proxy 经常会崩溃。这是因为它试图修改宿主机的内核参数（如加载内核模块、修改 sysctl），但即使是 privileged 容器，某些内核路径也是只读的或受限的。


操作 1：查看报错日志 请执行以下命令：
```shell
kubectl logs -n kube-system kube-proxy-4427s
```

原因分析： 日志显示 open /proc/sys/net/netfilter/nf_conntrack_max: permission denied。 即便你是特权容器（Privileged），在 Docker-in-Docker 环境中，kube-proxy 默认试图修改宿主机的内核连接跟踪参数（Conntrack）以优化性能，但因为 /proc 文件系统的隔离限制，这个操作被拒绝了。

解决方案： 我们要修改 kube-proxy 的配置，告诉它不要尝试修改内核参数（设为 0 即代表禁用修改）。

编辑 kube-proxy 的配置表： 在容器内执行：

```shell
kubectl -n kube-system edit configmap kube-proxy
```

修改 conntrack 字段： 在编辑器中找到 conntrack 部分（通常在文件中间位置），将 maxPerCore 和 min 都改为 0。

```shell
conntrack:
  # 【重点】改为 0，表示不修改内核参数
  maxPerCore: 0
  min: 0
  tcpCloseWaitTimeout: 1h0m0s
  ...
```

保存退出 (:wq)。

重启 kube-proxy 使配置生效：
```shell
kubectl -n kube-system delete pod -l k8s-app=kube-proxy
```

验证： 等待几秒后查看状态：
```shell
kubectl get pod -n kube-system | grep proxy
```


---

**打印默认配置参数到指定文件**
```bash
kubeadm config print init-defaults > kubeadm-init-example.yaml
```

```shell
kubeadm init --config kubeadm-init.yaml
```

指定配置文件方式初始化

```bash
kubeadm init --config kubeadm-init.yaml
```

```bash
kubectl get nodes
```

```bash
kubectl get pod -A
```

```bash
kubectl apply -f kube-flannel.yml
```

---


## 4 涉及问题

### 4.1 无法停止Docker服务

**背景**

- 本机：MacOS ARM64 M2
- 虚拟机：MacOS上通过Docker搭建的Ubuntu24.04LT作为虚拟机
- Docker: 在虚拟机中安装了Docker

**问题**

在Mac上启动的Docker容器中修改Docker配置，无法使用 `systemctl stop docker`停止服务。
```shell
# systemctl stop docker
Stopping 'docker.service', but its triggering units are still active:
docker.socket
```

**原因**

Docker 使用一种称为“Socket Activation”的机制。这意味着即使主服务停止了，它的“监听插口”（Socket）还在等待连接。如果有程序尝试连接 Docker，它可能会自动再次唤醒 Docker 服务，这会干扰我们的操作。

**解决方法**

同时停止 Socket 和 Service。

依次运行以下命令彻底停止它：

彻底停止 Docker
```shell
# 先停止 socket
sudo systemctl stop docker.socket

# 再停止 service (确保它真的停了)
sudo systemctl stop docker
```

运行完这两行后，就可以用 `sudo systemctl status docker` 确认一下，应该显示 inactive (dead)。

### 4.2 初始化集群失败

**背景**

- 环境：K8s in docker

**问题**

初始化集群出现下面错误：

```shell
# kind create cluster --name mycluster \
  --image m.daocloud.io/docker.io/kindest/node:v1.35.0
Creating cluster "mycluster" ...
 ✓ Ensuring node image (m.daocloud.io/docker.io/kindest/node:v1.35.0) 🖼 
 ✗ Preparing nodes 📦  
ERROR: failed to create cluster: command "docker run --name mycluster-control-plane --hostname mycluster-control-plane --label io.x-k8s.kind.role=control-plane --privileged --security-opt seccomp=unconfined --security-opt apparmor=unconfined --tmpfs /tmp --tmpfs /run --volume /var --volume /lib/modules:/lib/modules:ro -e KIND_EXPERIMENTAL_CONTAINERD_SNAPSHOTTER --detach --tty --label io.x-k8s.kind.cluster=mycluster --net kind --restart=on-failure:1 --init=false --cgroupns=private --publish=127.0.0.1:32865:6443/TCP -e KUBECONFIG=/etc/kubernetes/admin.conf m.daocloud.io/docker.io/kindest/node:v1.35.0" failed with error: exit status 125
Command Output: docker: Error response from daemon: failed to mount /tmp/containerd-mount4213980840: mount source: "overlay", target: "/tmp/containerd-mount4213980840", fstype: overlay, flags: 0, data: "workdir=/var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/3/work,upperdir=/var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/3/fs,lowerdir=/var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/2/fs:/var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/1/fs,index=off", err: invalid argument

Run 'docker run --help' for more information
```

****


---
**问题2**

vim /etc/containerd/config.toml
```shell
sandbox_image = "registry.aliyuncs.com/google_containers/pause:3.9"

# 1. 重置
kubeadm reset -f

# 2. 重新初始化
kubeadm init --config kubeadm-init.yaml --ignore-preflight-errors=all
```

**重新初始化**

```shell
kubeadm reset -f
rm -rf /etc/cni/net.d
rm -rf /var/lib/etcd
```

**解决问题**

```
# kubelet
I0123 09:24:28.723456     312 server.go:525] "Kubelet version" kubeletVersion="v1.35.0"
I0123 09:24:28.723756     312 server.go:527] "Golang settings" GOGC="" GOMAXPROCS="" GOTRACEBACK=""
I0123 09:24:28.723774     312 watchdog_linux.go:95] "Systemd watchdog is not enabled"
I0123 09:24:28.723779     312 watchdog_linux.go:138] "Systemd watchdog is not enabled or the interval is invalid, so health checking will not be started."
I0123 09:24:28.724337     312 server.go:676] "Standalone mode, no API client"
E0123 09:24:28.725833     312 run.go:72] "command failed" err="failed to run Kubelet: validate service connection: validate CRI v1 runtime API for endpoint \"unix:///run/containerd/containerd.sock\": rpc error: code = Unimplemented desc = unknown service runtime.v1.RuntimeService"

# 1. 备份并彻底重新生成
sudo cp /etc/containerd/config.toml /etc/containerd/config.toml.bak
containerd config default | sudo tee /etc/containerd/config.toml

# 2. 修正关键配置 (使用 sed 自动化修改)
# 确保没有禁用 CRI
sudo sed -i 's/disabled_plugins = \["cri"\]/disabled_plugins = []/g' /etc/containerd/config.toml

# 开启 SystemdCgroup
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml

# 3. 重启
sudo systemctl restart containerd
```

---
标准做法
```
sudo cp /etc/containerd/config.toml /etc/containerd/config.toml.bak
containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i 's/disabled_plugins = \["cri"\]/disabled_plugins = []/g' /etc/containerd/config.toml
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
sudo systemctl restart containerd
```

```
crictl pull registry.aliyuncs.com/google_containers/pause:3.10.1
    --service-cidr=10.1.0.0/16 \

journalctl -xeu kubelet | tail -n 30
```



---

```bash
# kubectl get pod -A
NAMESPACE      NAME                           READY   STATUS             RESTARTS        AGE
kube-flannel   kube-flannel-ds-7rnw9          1/1     Running            5 (77m ago)     82m
kube-flannel   kube-flannel-ds-pgth6          0/1     CrashLoopBackOff   8 (5m2s ago)    25m
kube-flannel   kube-flannel-ds-qwp78          1/1     Running            0               49m
kube-system    coredns-bbdc5fdf6-ch5s8        1/1     Running            0               57m
kube-system    coredns-bbdc5fdf6-gvn7s        1/1     Running            0               57m
kube-system    etcd-vm-1                      1/1     Running            0               107m
kube-system    kube-apiserver-vm-1            1/1     Running            0               107m
kube-system    kube-controller-manager-vm-1   1/1     Running            0               107m
kube-system    kube-proxy-f5jrv               1/1     Running            0               49m
kube-system    kube-proxy-h6zw6               0/1     CrashLoopBackOff   9 (4m49s ago)   25m
kube-system    kube-proxy-vjxfv               1/1     Running            0               76m
kube-system    kube-scheduler-vm-1            1/1     Running            0               107m

# kubectl logs -n kube-system kube-proxy-h6zw6
E0128 04:02:58.847030       1 run.go:72] "command failed" err="failed complete: too many open files"

# kubectl get po -A -o wide
NAMESPACE      NAME                           READY   STATUS             RESTARTS         AGE    IP            NODE   NOMINATED NODE   READINESS GATES
kube-flannel   kube-flannel-ds-7rnw9          1/1     Running            5 (84m ago)      90m    172.20.30.1   vm-1   <none>           <none>
kube-flannel   kube-flannel-ds-pgth6          0/1     CrashLoopBackOff   10 (80s ago)     33m    172.20.30.3   vm-3   <none>           <none>
kube-flannel   kube-flannel-ds-qwp78          1/1     Running            0                57m    172.20.30.2   vm-2   <none>           <none>
kube-system    coredns-bbdc5fdf6-ch5s8        1/1     Running            0                65m    10.244.0.4    vm-1   <none>           <none>
kube-system    coredns-bbdc5fdf6-gvn7s        1/1     Running            0                65m    10.244.0.5    vm-1   <none>           <none>
kube-system    etcd-vm-1                      1/1     Running            0                114m   172.20.30.1   vm-1   <none>           <none>
kube-system    kube-apiserver-vm-1            1/1     Running            0                114m   172.20.30.1   vm-1   <none>           <none>
kube-system    kube-controller-manager-vm-1   1/1     Running            0                114m   172.20.30.1   vm-1   <none>           <none>
kube-system    kube-proxy-f5jrv               1/1     Running            0                57m    172.20.30.2   vm-2   <none>           <none>
kube-system    kube-proxy-h6zw6               0/1     CrashLoopBackOff   11 (2m10s ago)   33m    172.20.30.3   vm-3   <none>           <none>
kube-system    kube-proxy-vjxfv               1/1     Running            0                83m    172.20.30.1   vm-1   <none>           <none>
kube-system    kube-scheduler-vm-1            1/1     Running            0                114m   172.20.30.1   vm-1   <none>           <none>

# ulimit -n
1024
```

```
你可以直接复制以下命令在终端执行。这套命令会先备份原文件，然后将配置追加到 `/etc/security/limits.conf` 文件末尾。

### 1. 备份原文件（安全起见）

```bash
sudo cp /etc/security/limits.conf /etc/security/limits.conf.bak

```

### 2. 写入配置 (使用 `tee` 追加)

这条命令会将软限制和硬限制都设置为 65535，且对所有用户生效：

```bash
echo -e "\n* soft nofile 65535\n* hard nofile 65535" | sudo tee -a /etc/security/limits.conf

```

### 3. 验证是否写入成功

运行以下命令，如果能看到刚才写入的两行内容，说明操作成功：

```bash
tail -n 2 /etc/security/limits.conf

```

---

### ⚠️ 重要提示：使其生效

修改该文件后，系统**不会立即生效**。你需要执行以下任一操作：

1. **最彻底：** 重启服务器 (`reboot`)。
2. **最快：** 断开当前的 SSH 连接，**重新登录**。

重新登录后，输入 `ulimit -n` 检查，如果显示 `65535`，即代表修改完成。
```

---

```bash
sudo cp /etc/security/limits.conf /etc/security/limits.conf.bak

sudo tee -a /etc/security/limits.conf > /dev/null <<EOF

# --- Custom Production Configuration Start ---
root soft nofile 655350
root hard nofile 655350
* soft nofile 655350
* hard nofile 655350
* soft stack 20480
* hard stack 20480
* soft nproc 655360
* hard nproc 655360
* soft core unlimited
* hard core unlimited
# --- Custom Production Configuration End ---
EOF
```

vim /etc/security/limits.conf
内容如下：
root soft nofile 655350
root hard nofile 655350
* soft nofile 655350
* hard nofile 655350
* soft stack 20480
* hard stack 20480
* soft nproc 655360
* hard nproc 655360
* soft core unlimited
* hard core unlimited
退出当前会话，重新登录。执行以下命令，查看配置是否生效：
ulimit -a



---

### 3.7 （可选）解决coredns节点未启动问题

**只差最后一步了！** CoreDNS 的 `CrashLoopBackOff` 在这种自定义容器（Docker-in-Docker）环境中，99% 的原因都是 **“DNS 循环解析（DNS Loop）”**。

简单来说：CoreDNS 启动后，发现宿主机（`vm-1` 容器）的 DNS 指向了本地（比如 `127.0.0.53`），而 CoreDNS 自己也监听本地，导致它认为“如果不停止，查询就会陷入死循环”，于是它主动崩溃退出。

请按顺序执行以下诊断和修复：

1、第一步：确认病因（必做）

查看报错日志，验证我的猜测：

```bash
# kubectl logs -n kube-system coredns-bbdc5fdf6-2qnqb
maxprocs: Leaving GOMAXPROCS=4: CPU quota undefined
.:53
[INFO] plugin/reload: Running configuration SHA512 = 1b226df79860026c6a52e67daa10d7f0d57ec5b023288ec00c5e05f93523c894564e15b91770d3a07ae1cfbe861d15b37d4a0027e69c546ab112970993a3b03b
CoreDNS-1.13.1
linux/amd64, go1.25.2, 1db4568
[FATAL] plugin/loop: Loop (127.0.0.1:53177 -> :53) detected for zone ".", see https://coredns.io/plugins/loop#troubleshooting. Query: "HINFO 3066367716020298911.5945620039662324039."
```

**如果看到类似下面的日志：**

> `plugin/loop: Loop (127.0.0.1:xxxxx -> :53) detected for zone "."`

说明确实是 **DNS Loop** 问题。


2、第二步：修复方案（二选一）

方案 A：修改宿主机 DNS（推荐，最彻底）

CoreDNS 默认继承 `vm-1` 容器的 `/etc/resolv.conf`。如果这个文件里写的是 `nameserver 127.0.0.53`（Ubuntu 默认），就会导致循环。我们将它改为外部 DNS 即可。

**在 `vm-1` 容器内执行：**

```bash
# 1. 备份原文件
cp /etc/resolv.conf /etc/resolv.conf.bak

# 2. 强制修改为阿里云 DNS 或 Google DNS
# (注意：如果是软链接，可能需要先 rm /etc/resolv.conf 再创建)
echo "nameserver 223.5.5.5" > /etc/resolv.conf
# 或者
echo "nameserver 8.8.8.8" > /etc/resolv.conf

# 3. 删除 CoreDNS Pod 让它自动重启
kubectl delete pod -n kube-system -l k8s-app=kube-dns
```

*等待几秒，新的 Pod 启动时会读取到新的 DNS 配置，就不会报错了。*

```bash
# kubectl get pod -A
NAMESPACE      NAME                           READY   STATUS    RESTARTS      AGE
kube-flannel   kube-flannel-ds-7rnw9          1/1     Running   5 (20m ago)   25m
kube-system    coredns-bbdc5fdf6-ch5s8        1/1     Running   0             12s
kube-system    coredns-bbdc5fdf6-gvn7s        1/1     Running   0             12s
kube-system    etcd-vm-1                      1/1     Running   0             50m
kube-system    kube-apiserver-vm-1            1/1     Running   0             50m
kube-system    kube-controller-manager-vm-1   1/1     Running   0             50m
kube-system    kube-proxy-vjxfv               1/1     Running   0             18m
kube-system    kube-scheduler-vm-1            1/1     Running   0             50m
```

---

方案 B：修改 CoreDNS 配置（移除循环检测）

如果你不想改 `/etc/resolv.conf`，可以通过修改 K8s 配置，告诉 CoreDNS：“别检查循环了，直接跑”。

**在 `vm-1` 容器内执行：**

1. **编辑 ConfigMap**
```bash
kubectl edit cm coredns -n kube-system
```

2. **删除 `loop` 插件**
找到 `Corefile` 部分，你会看到类似这样的内容：
```yaml
.:53 {
    errors
    health {
       lameduck 5s
    }
    ready
    kubernetes cluster.local in-addr.arpa ip6.arpa {
       pods insecure
       fallthrough in-addr.arpa ip6.arpa
       ttl 30
    }
    prometheus :9153
    forward . /etc/resolv.conf {
       max_concurrent 1000
    }
    cache 30
    loop  # <--- 【找到这一行，删除它！】
    reload
    loadbalance
}
```


**删掉 `loop` 这一行**，然后保存退出 (`:wq`)。
3. **重启 CoreDNS Pod**
```bash
kubectl delete pod -n kube-system -l k8s-app=kube-dns
```

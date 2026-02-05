+++
title = 'kueadm安装过程'
date = 2026-01-28T17:40:09+08:00
draft = true
categories = [ "Programming" ]
tags = [ "k8s", "kubernetes" ]
+++

## Kubeadm安装原理

```bash
# kubectl get nodes
NAME   STATUS   ROLES           AGE   VERSION
vm-1   Ready    control-plane   86m   v1.35.0
vm-2   Ready    <none>          79m   v1.35.0
vm-3   Ready    <none>          78m   v1.35.0
```

现在已经搭建了一个3节点的K8s集群，Master(vm-1)节点不会处理任何Pod、WorkNode资源相关的任务，这些任务会被调度到Work Node节点上。

`kubeadm init ...`执行之后会发生一系列事情，这也是K8s集群部署时非常容易出错，一方面是init流程很长，需要配合前期的一些准备才能顺利安装成功。

Init 命令的工作流程  
kubeadm init 命令通过执行下列步骤来启动一个 Kubernetes Control Plane 节点。

1. 首先会运行一系列的预检项来验证系统状态。

比如硬件要求、Cgroup Driver要求等。一些检查项目仅仅触发警告，其它的则会被视为错误并且退出 kubeadm，除非问题得到解决或者用户指定了 --ignore-preflight-errors= 参数。

2. 预检查都通过后会生成一个自签名的 CA 证书（或者使用现有的证书，如果提供的话）来为集群中的每一个组件建立身份标识。如果用户已经通过 --cert-dir 配置的证书目录（默认为 /etc/kubernetes/pki）提供了他们自己的 CA 证书以及/或者密钥，那么将会跳过这个步骤，正如文档使用自定义证书所述。

每个组件都有相关证书和密钥，除了自生成的也可以自行提供证书。

```bash
# ll /etc/kubernetes/pki/
apiserver.crt
apiserver-etcd-client.crt
apiserver-etcd-client.key
apiserver.key
apiserver-kubelet-client.crt
apiserver-kubelet-client.key
ca.crt
ca.key
etcd/
front-proxy-ca.crt
front-proxy-ca.key
front-proxy-client.crt
front-proxy-client.key
sa.key
sa.pub
```

3. 生成证书完成之后会将`kubeconfig`文件写入`/etc/kubernetes/`目录，以便kubelet、控制器管理器和调度器用来连接到 API 服务器，它们每一个都有自己的身份标识，同时生成一个名为`admin.conf`的独立的kubeconfig文件，用于管理操作。admin.conf也会从Master节点拷贝到Work Node节点上。

在执行 `kubeadm reset` 命令后这些文件也会被清理。

4. 接着为 API 服务器、控制器管理器和调度器生成静态 Pod 的清单文件。静态 Pod 的清单文件被写入到 /etc/kubernetes/manifests 目录；kubelet 会监视这个目录以便在系统启动的时候创建 Pod。一旦 Control Plane 的 Pod 都运行起来，kubeadm init 的工作流程就继续往下执行。

5. 对 Control Plane 节点应用 labels 和 taints 标记以便不会在它上面运行其它的工作负载。

6. 生成令牌以便其它节点以后可以使用这个令牌向 Control Plane 节点注册自己。

7. Kubeadm 会创建 configmap，提供添加节点所需要的信息。 
```bash
# kubectl get configmap -n kube-system
NAME                                                   DATA   AGE
coredns                                                1      105m
extension-apiserver-authentication                     6      105m
kube-apiserver-legacy-service-account-token-tracking   1      105m
kube-proxy                                             2      105m
kube-root-ca.crt                                       1      105m
kubeadm-config                                         1      105m
kubelet-config                                         1      105m

# # kubectl describe configmap kubeadm-config -n kube-system
Name:         kubeadm-config
Namespace:    kube-system
Labels:       <none>
Annotations:  <none>

Data
====
ClusterConfiguration:
----
apiServer: {}
apiVersion: kubeadm.k8s.io/v1beta4
caCertificateValidityPeriod: 87600h0m0s
certificateValidityPeriod: 8760h0m0s
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns: {}
encryptionAlgorithm: RSA-2048
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: registry.aliyuncs.com/google_containers
kind: ClusterConfiguration
kubernetesVersion: v1.35.0
networking:
  dnsDomain: cluster.local
  podSubnet: 10.244.0.0/16
  serviceSubnet: 10.96.0.0/12
proxy: {}
scheduler: {}



BinaryData
====

Events:  <none>
```


+++
title = '创建一个简单的Pod'
date = 2026-01-31T21:21:26+08:00
draft = true
categories = [ "Programming" ]
tags = [ "k8s", "kubernetes" ]
+++

## Nginx Pod

my-nginx.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
	name: my-nginx
  labels:
	  name: my-nginx

spec:
	containers:
	- name: my-nginx
		image: docker.io/Library/nginx:latest
		resources:
			limits:
				memory: "128Mi"
				cpu: "500M"
		ports:
		- containerPort: 8081  
```

上传至node1节点并执行：
```shell
kubectl create -f nginx.yaml
```
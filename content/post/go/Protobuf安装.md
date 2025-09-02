+++
title = 'Protobuf安装'
date = 2025-02-06T11:34:31+08:00
draft = false
categories = [ "Go" ]
tags = [ "go", "protobuf" ]
+++

介绍 `Protobuf` 的两种安装方式。

## 编译安装

参考：[mac 安装protobuf步骤](https://zhuanlan.zhihu.com/p/60471892)

1、从github上下载protobuf3，这里以 Mac 版本 [Protocol Buffers v3.13.0](https://github.com/protocolbuffers/protobuf/releases/tag/v3.13.0) 为例。有很多语言版本，Mac版本选择第一个。

![](/images/protobuf/10.jpg)

2、解压并进入目录

```bash
sudo tar -zxvf protobuf-all-3.13.0.tar.gz
cd protobuf-3.13.0/
```

3、设置安装目录

```bash
sudo ./configure --prefix=/usr/local/protobuf
```

4、安装

```bash
sudo make
sudo make install
```

5、配置环境变量，编辑 `.zshrc` 或者 `.bash_profile` 等文件，添加下面两行

```bash
export PROTOBUF=/usr/local/protobuf 
export PATH=$PROTOBUF/bin:$PATH
```

6、`source` 一下使文件生效

```bash
source ~/.zshrc
```

或者

```bash
source .bash_profile
```

7、测试安装结果

```bash
protoc --version
```

![](/images/protobuf/20.png)


## 包管理器安装

**Linux**

```bash
apt install -y protobuf-compiler
protoc --version
```

**MacOS**

```bash
brew install protobuf
protoc --version
```
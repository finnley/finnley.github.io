+++
title = 'Quick Start'
date = 2024-10-17T10:02:41+08:00
draft = true
+++

# docker-build

## Centos altarch 7


### 介绍
该镜像针对我本地笔记本所在的开发学习环境（Apple M2）所使用的镜像。

相关配置链接：
[Centos altarch镜像](https://developer.aliyun.com/mirror/centos-altarch?spm=a2c6h.13651102.0.0.534d1b11CbZMrv)

### Dockerfile

```bash
# 基础镜像
FROM centos:centos7.9.2009

# 
RUN mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup \
        && curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-altarch-7.repo \
        && yum clean all \
        && yum makecache

# MySQL 依赖以及常用工具
RUN yum install -y libaio numactl perl
# openssh-server: sshd
RUN yum install -y sudo vim lsof wget which vim git iproute net-tools openssh-server \
        && yum clean all

RUN echo root:sshpass | chpasswd \
        && cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
        && chmod u+s $(which ping) \
        && localedef -i zh_CN -f UTF-8 zh_CN.UTF-8 \
        && localedef -i en_US -f UTF-8 en_US.UTF-8
ENV LC_ALL=zh_CN.UTF-8

#RUN (cd /lib/systemd/system/sysinit.target.wants/; for i in *; do [ $i == systemd-tmpfiles-setup.service ] || rm -f $i; done) && \
#    rm -f /lib/systemd/system/multi-user.target.wants/* && \
#    rm -f /etc/systemd/system/*.wants/* && \
#    rm -f /lib/systemd/system/local-fs.target.wants/* && \
#    rm -f /lib/systemd/system/sockets.target.wants/*udev* && \
#    rm -f /lib/systemd/system/sockets.target.wants/*initctl* && \
#    rm -f /lib/systemd/system/basic.target.wants/* && \
#    rm -f /lib/systemd/system/anaconda.target.wants/*

# Docker 安装
# 参考：https://developer.aliyun.com/mirror/docker-ce?spm=a2c6h.13651102.0.0.4ae91b11pHpBJN&userCode=okjhlpr5
RUN sudo yum install -y yum-utils device-mapper-persistent-data lvm2
RUN sudo yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
RUN sudo sed -i 's+download.docker.com+mirrors.aliyun.com/docker-ce+' /etc/yum.repos.d/docker-ce.repo
RUN sudo yum makecache fast
RUN sudo yum -y install docker-ce
RUN systemctl enable docker
RUN cat > /etc/docker/daemon.json <<EOF
{
    "registry-mirrors": [
        "https://mirror.ccs.tencentyun.com",
        "https://docker.mirrors.ustc.edu.cn",
        "https://vfs1y0dl.mirror.aliyuncs.com",
        "https://docker.mirrors.ustc.edu.cn",
        "https://registry.docker-cn.com"
    ]
}
EOF
# RUN systemctl daemon-reload && systemctl restart docker
# RUN systemctl start docker

COPY ../* /docker-build/

VOLUME [ "/sys/fs/cgroup" ]
CMD ["/usr/sbin/init"]

```


就感觉我们（王林）将要化神，你（周武泰）却想在我们身上种下破坏我们化神的种子，影响我们成功化神。
**基础镜像**

“FROM centos:centos7.9.2009“ 表示基础镜像构建是基于 centos:centos7.9.2009。

该镜像可以从 [DockerHub](https://hub.docker.com/layers/library/centos/centos7.9.2009/images/sha256-dead07b4d8ed7e29e98de0f4504d87e8880d4347859d839686a31da35a3b532f?context=explore) 上获取。

也可以访问 [阿里云镜像站](https://developer.aliyun.com/mirror/?spm=a2c6h.25603864.0.0.2a5c7c65j9T7qU)，搜索 `centos` 找到镜像。


**设置镜像源**

参考[Centos7配置阿里云yum源](https://developer.aliyun.com/article/921275)

1、备份系统原来的repo文件
```bash
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
```


2、下载阿里云 repo 文件
```bash
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-altarch-7.repo
```

3、更新yum源
```bash
yum clean all
yum makecache
yum update -y 
```

**安装 MySQL 依赖**

参考：[2.2 Installing MySQL on Unix/Linux Using Generic Binaries](https://dev.mysql.com/doc/refman/8.4/en/binary-installation.html)

```bash
yum install -y libaio numactl perl
```

**常用工具**

```bash
# openssh-server: sshd
RUN yum install -y sudo vim lsof wget which vim git iproute net-tools openssh-server \
        && yum clean all
```

- openssh-server 用于 sshd 连接

**修改root ssh密码**

```bash
echo root:sshpass | chpasswd
```

这条命令 `echo root:sshpass | chpasswd` 的作用如下：
`echo root:sshpass` 打印出 `root:sshpass` 这个字符串。
`|` 表示管道，将 echo 命令输出的结果传递给下一个命令。
`chpasswd` 是用来批量修改用户密码。

因此，这条命令的意思是将用户名为 root 的用户的密码修改为 sshpass。其中 root:sshpass 表示用户名和密码之间使用 : 分隔。当这条命令被执行后，用户 root 的密码将会被设置为 sshpass。请注意，使用明文密码作为参数传递可能存在安全风险，建议谨慎使用。

**Docker安装**

参考：[Docker CE镜像](https://developer.aliyun.com/mirror/docker-ce?spm=a2c6h.13651102.0.0.4ae91b11pHpBJN&userCode=okjhlpr5)

```bash
RUN sudo yum install -y yum-utils device-mapper-persistent-data lvm2
RUN sudo yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
RUN sudo sed -i 's+download.docker.com+mirrors.aliyun.com/docker-ce+' /etc/yum.repos.d/docker-ce.repo
RUN sudo yum makecache fast
RUN sudo yum -y install docker-ce
RUN systemctl enable docker
RUN cat > /etc/docker/daemon.json <<EOF
{
    "registry-mirrors": [
        "https://mirror.ccs.tencentyun.com",
        "https://docker.mirrors.ustc.edu.cn",
        "https://vfs1y0dl.mirror.aliyuncs.com",
        "https://docker.mirrors.ustc.edu.cn",
        "https://registry.docker-cn.com"
    ]
}
EOF
# RUN systemctl daemon-reload && systemctl restart docker
# RUN systemctl start docker

# 将 entrypoint.sh 复制到镜像中
COPY entrypoint.sh /usr/local/bin/entrypoint.sh
RUN chmod +x /usr/local/bin/entrypoint.sh

...

# 使用 entrypoint.sh 作为入口点
ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]
```

在构建镜像我本想启动时执行上面两行注释掉的命令，发现这个命令并没有生效，所以我在dcoker-build 目录下新增了一个 entrypoint.sh 脚本，脚本内容如下：
```bash
#!/bin/bash
set -e

# 重新加载系统服务
systemctl daemon-reload
# 重启 docker 服务
systemctl restart docker

# 保持容器运行
exec "$@"
```

- 使用 `set -e` 确保任何命令失败时，脚本将退出。
- `exec "$@"` 将实际传递给容器的命令替换为脚本本身运行的命令，这样可以保持信号的传递和执行环境的一致性。
- 如果你的容器需要保持运行，可以确保有一个长期运行的进程（比如某个服务），否则容器会在脚本执行完成后退出。

将那两条命令放入一个 shell 脚本中，然后使用 `CMD` 或 `ENTRYPOINT` 命令来运行这个脚本。


**拷贝镜像构建目录**

```bash
COPY ../* /docker-build/
```

**挂载cgroup**

```bash
VOLUME [ "/sys/fs/cgroup" ]
```

**启动**

```bash
CMD ["/usr/sbin/init"]
```



### 小结

docker run -itd --privileged --name=udp-9 reg.einscat.com:10010/library/centos7:altarch

export DOCKER_BUILDKIT=1
export COMPOSE_DOCKER_CLI_BUILD=1



在 Dockerfile 中，`RUN` 命令是在构建镜像时执行的，而不是在运行容器时执行的。如果要在容器启动时执行命令，应该使用 `CMD` 或 `ENTRYPOINT`。

要在启动容器时执行 `systemctl daemon-reload` 和 `systemctl restart docker`，可以将这些命令放入一个 shell 脚本中，然后使用 `CMD` 或 `ENTRYPOINT` 命令来运行这个脚本。

下面是一个修改后的示例，展示如何实现这一点：

1. 创建一个 shell 脚本，例如 `entrypoint.sh`：

```bash
#!/bin/bash
set -e

# 重新加载系统服务
systemctl daemon-reload
# 重启 docker 服务
systemctl restart docker

# 保持容器运行
exec "$@"
```

2. 在 Dockerfile 中使用 `COPY` 命令将该脚本复制到镜像中，并设置合适的权限：

```dockerfile
# 其他 Dockerfile 内容...

# 将 entrypoint.sh 复制到镜像中
COPY entrypoint.sh /usr/local/bin/entrypoint.sh
RUN chmod +x /usr/local/bin/entrypoint.sh

# 使用 entrypoint.sh 作为入口点
ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]
CMD ["your_default_command_if_needed"]  # 可以替换为实际的默认命令
```

请注意以下几点：

- 使用 `set -e` 确保任何命令失败时，脚本将退出。
- `exec "$@"` 将实际传递给容器的命令替换为脚本本身运行的命令，这样可以保持信号的传递和执行环境的一致性。
- 如果你的容器需要保持运行，可以确保有一个长期运行的进程（比如某个服务），否则容器会在脚本执行完成后退出。

以上示例中，`your_default_command_if_needed` 可以根据你的需求进行替换或省略。如果你只需要执行重启命令且不需要处理其他命令，可以直接在 `CMD` 中省略。
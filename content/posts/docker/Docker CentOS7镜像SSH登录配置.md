+++
title = 'Docker CentOS7镜像SSH登录配置.md'
date = 2025-11-20T21:38:21+08:00
draft = true
categories = [ "Docker" ]
tags = [ "docker" ]
+++

# 脚本分析

```Dockerfile
RUN echo "root:${ROOT_PASSWORD}" | chpasswd && \
    ln -sf /usr/share/zoneinfo/${TIMEZONE} /etc/localtime && \
    chmod u+s $(which ping) && \
    localedef -i zh_CN -f UTF-8 zh_CN.UTF-8 || true
```

这段脚本通常出现在 **ARM64 架构的 CentOS 7 镜像（比如华为云、阿里云、腾讯云的鲲鹏/飞腾服务器）的一键初始化或 Dockerfile/云镜像自动化脚本**里，目的是快速把一个“裸系统”配置成可直接 SSH 登录使用的状态。

我们逐行解释这四条命令到底在干什么：

```bash
echo "root:${ROOT_PASSWORD}" | chpasswd && \
```

- 用变量 `${ROOT_PASSWORD}`（前面必须已经定义，比如 ROOT_PASSWORD=123456）来**批量修改 root 用户的密码**。
- 相当于手动执行：`echo 'root:123456' | chpasswd`
- 这样就不用交互式输入 `passwd root` 两次了，常用于自动化部署、Dockerfile、云 init 脚本。
- 举例：如果 ROOT_PASSWORD=Abc@123，这条命令执行完后，root 密码立刻变成 Abc@123。


```bash
ln -sf /usr/share/zoneinfo/${TIMEZONE} /etc/localtime && \
```
- 把时区设置为环境变量 `${TIMEZONE}` 指定的时区（常见值：Asia/Shanghai）。
- 例如 TIMEZONE=Asia/Shanghai，就会执行：
  `ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime`
- 这样系统时间就变成中国时间（CST，北京时间），不再是默认的 UTC。

```bash
chmod u+s $(which ping) && \
```
- 给 `ping` 命令加上 **setuid 位**，让普通用户也能 ping 成功。
- 为什么需要这一步？
  - 在 CentOS 7 aarch64（AltArch）某些精简镜像里，ping 默认没有 setuid，普通用户执行 ping 会报错：
    `ping: socket: Operation not permitted`
  - 因为 ping 需要创建 raw socket，只有 root 才有权限。
  - 加了 `u+s` 后，ping 程序会以 root 权限运行，哪怕普通用户也能正常 ping（这是 Linux 几十年的传统做法）。
  - 执行后你会看到：
    ```bash
    -rwsr-xr-x. 1 root root ... /bin/ping   # 注意 s 字母
    ```

```bash
localedef -i zh_CN -f UTF-8 zh_CN.UTF-8 || true
```
- 强制生成中文（zh_CN.UTF-8）语言环境。
- 很多云厂商的 CentOS 7 ARM 镜像默认只装了 C/POSIX 环境，没有中文 locale，导致执行中文相关命令会报错：
  `locale: Cannot set LC_CTYPE to default locale: No such file or directory`
- 这条命令就是补上中文语言包。
- 后面的 `|| true` 是为了防止某些系统已经存在了再生成时报错，整个脚本继续往下走。

### 总结：这四条命令整体作用就是

把一台刚装好的 CentOS 7（aarch64）服务器快速变成：

1. root 有密码，可以直接 SSH 登录（不用密钥）
2. 时间是中国时间
3. 普通用户也能 ping（网络排查方便）
4. 支持中文显示，不报 locale 错误

这四条几乎是所有**国产 ARM 服务器（鲲鹏、飞腾等）CentOS 7 镜像的标配初始化脚本**，你会在华为云、阿里云、腾讯云的官方镜像 marketplace 里经常看到。

一句话概括：
**“把裸 CentOS 7 ARM 服务器 30 秒变成可直接上手用的状态”**。
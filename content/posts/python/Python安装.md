+++
title = 'Python安装'
date = 2020-01-02T08:10:11+08:00
draft = true
categories = [ "Python" ]
tags = [ "python" ]
+++

# 1 Linux源码编译安装

1、安装依赖。

```shell
# CentOS
sudo yum install -y openssl-devel bzip2-devel expat-devel gdbm-devel readline-devel sqlite-devel gcc gcc-c++  openssl-devel libffi-devel mariadb-devel xz-devel tk-devel tcl-devel libuuid-devel
# Debian/Ubuntu
sudo apt-get install -y zlib1g-dev libbz2-dev libssl-dev libncurses5-dev default-libmysqlclient-dev libsqlite3-dev libreadline-dev tk-dev libgdbm-dev libdb-dev libpcap-dev xz-utils libexpat1-dev liblzma-dev libffi-dev libc6-dev tk-dev tcl-dev uuid-dev
```

2、下载。

```shell
wget https://www.python.org/ftp/python/3.13.0/Python-3.13.0.tar.xz
```

3、解压，将 Python 压缩包解压至 /tmp 目录下。

```shell
tar xvf Python-3.13.0.tar.xz -C /tmp
```

4、编译，把 `Python3.13` 安装到 `/usr/local` 目录。

```shell
cd /tmp/Python-3.13.0/
./configure --enable-optimizations --prefix=/usr/local/python3.13
make -j 4
```

- 切换当前工作目录到 /tmp/Python-3.13.0/，后续操作将在此目录中进行，然后配置 Python 的构建环境，生成编译所需的 Makefile 文件。
- `--enable-optimizations`: 启用优化选项，优化 Python 的性能（如启用 PGO 和 LTO，Profile-Guided Optimization 和 Link-Time Optimization）。
- `-prefix=/usr/local/python3.13`: 指定 Python 的安装路径为 /usr/local/python3.13，避免覆盖系统默认的 Python 版本。
- `make`: 根据 configure 生成的 Makefile，执行编译过程。
- `-j 4`: 使用 4 个并行任务来加速编译过程。数字 4 通常与 CPU 核心数相关，适合 4 核 CPU，可以根据实际硬件调整（例如，8 核 CPU 可以用 -j 8）。

如果出现 `bash: make: command not found`，需要安装编译工具：

```shell
# CentOS
sudo yum install -y gcc gcc-c++ automake autoconf libtool make
# Debian/Ubuntu
sudo apt-get install -y gcc automake autoconf libtool make
```

5、安装。

```shell
make altinstall
...
Successfully installed pip-24.2
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager, possibly rendering your system unusable.It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv. Use the --root-user-action option if you know what you are doing and want to suppress this warning.
```

- 编译完成后，将 Python 3.13.0 安装到之前 ./configure 中指定的路径（即 /usr/local/python3.13）。
- 与 make install 不同，make altinstall 是一种“替代安装”方式，避免覆盖系统默认的 Python 可执行文件（如 /usr/bin/python3）。


6、设置软链。

当输入 `python3` 回车后提示下面内容：

```shell
# python3
bash: python3: command not found
```

设置链接：

```shell
ln -s /usr/local/python3.13/bin/python3.13 /usr/bin/python3
ln -s /usr/local/python3.13/bin/pip3.13 /usr/bin/pip3
```

然后再输入 `python3` 查看结果：

```shell
# python3
Python 3.13.0 (main, Nov  3 2024, 17:31:54) [GCC 8.5.0 20210514 (Red Hat 8.5.0-4)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>>
```

# 使用uv安装Python

1、首先安装`uv`。

- [uv文档](https://docs.astral.sh/uv/)


# 3 更新日志

- 2024.11.04更新Python版本为3.13。
- 2025.12.22新增Python的uv安装方式。

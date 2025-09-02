+++
title = 'Python安装与配置'
date = 2020-01-02T13:10:11+08:00
draft = false
categories = [ "Python" ]
tags = [ "python" ]
+++

## 1 源码编译安装

### 1.1 安装依赖

CentOS：
```bash
sudo yum install -y openssl-devel bzip2-devel expat-devel gdbm-devel readline-devel sqlite-devel gcc gcc-c++  openssl-devel libffi-devel mariadb-devel xz-devel tk-devel tcl-devel libuuid-devel
```

Debian/Ubuntu：
```bash
sudo apt-get install -y zlib1g-dev libbz2-dev libssl-dev libncurses5-dev default-libmysqlclient-dev libsqlite3-dev libreadline-dev tk-dev libgdbm-dev libdb-dev libpcap-dev xz-utils libexpat1-dev liblzma-dev libffi-dev libc6-dev tk-dev tcl-dev uuid-dev
```

### 1.2 下载
```bash
wget https://www.python.org/ftp/python/3.13.0/Python-3.13.0.tar.xz
```

### 1.3 解压
```bash
tar xvf Python-3.13.0.tar.xz -C /tmp
```

### 1.4 编译

把 `Python3.8` 安装到 `/usr/local` 目录：
```bash
cd /tmp/Python-3.13.0/
./configure --enable-optimizations --prefix=/usr/local/python3.13
make -j 4
```

如果出现 `bash: make: command not found`，需要安装编译工具：

```bash
# CentOS
sudo yum install -y gcc gcc-c++ automake autoconf libtool make
# Debian/Ubuntu
sudo apt-get install -y gcc automake autoconf libtool make
```

### 1.5 安装
```bash
make altinstall
...
Successfully installed pip-24.2
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager, possibly rendering your system unusable.It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv. Use the --root-user-action option if you know what you are doing and want to suppress this warning.
```

### 1.6 设置软链

当输入 `python3` 回车后提示下面内容：
```bash
[root@chaos-1 Python-3.13.0]# python3
bash: python3: command not found
```

设置链接：
```bash
ln -s /usr/local/python3.13/bin/python3.13 /usr/bin/python3
ln -s /usr/local/python3.13/bin/pip3.13 /usr/bin/pip3
```

然后再输入 `python3` 查看结果：
```bash
[root@chaos-1 Python-3.13.0]# python3
Python 3.13.0 (main, Nov  3 2024, 17:31:54) [GCC 8.5.0 20210514 (Red Hat 8.5.0-4)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>>
```

## 2 Python 虚拟环境安装与配置

**参考**
- [Python学习：解决pip总是WARNING: Retrying (Retry(total=4...](https://blog.csdn.net/bjtu_linxinyu/article/details/105468658)

### 2.1 Windows

1、安装。安装完之后会在 Python 的可执行路径下安装一个 virtualenv 的 exe 文件
```bash
pip install virtualenvwrapper-win
```

2、使用 `mkvirtualenv` 命令创建虚拟环境，安装的时候会默认基于 Python 默认版本去新建一个虚拟环境
```bash
mkvirtualenv python_start
```

3、删除虚拟环境
```bash
rmvirtualenv python_start
```

创建的虚拟环境是默认 python 版本，也可以通过 `-p` 参数指定其他 Python 版本在创建虚拟环境：
```bash
mkvirtualenv -p {Python Path}\python.exe python_start
```

### 2.2 Linux

1、安装。如果下载比较慢，可以使用镜像
```bash
pip3 install virtualenv -i https://pypi.tuna.tsinghua.edu.cn/simple
pip3 install virtualenvwrapper -i https://pypi.tuna.tsinghua.edu.cn/simple
```

2、配置

找到 `virtualenvwrapper.sh` 文件。这里不同的 Linux 系统 `virtualenvwrapper.sh` 路径可能不一致，最好通过 `find` 命令查询一下。
```bash
# CentOS
find / -name 'virtualenvwrapper.sh'
/usr/local/python3.13/bin/virtualenvwrapper.sh

# MacOS
/Library/Frameworks/Python.framework/Versions/3.13/bin/virtualenvwrapper.sh
```

编辑 `.bashrc` 文件：  
```bash
vim ~/.bashrc
```

在最后添加下面内容：
```bash
export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3
export WORKON_HOME=$HOME/.virtualenvs
export VIRTUALENVWRAPPER_VIRTUALENV=/usr/local/python3.13/bin/virtualenv
source /usr/local/python3.13/bin/virtualenvwrapper.sh
```

- `VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3` 这句话表示当使用 `virtualenv` 去新建 python 虚拟环境的时候会到 `/usr/bin/python3` 位置去找 python 的可执行文件。

最后执行下面命令重新加载：
```bash
source  ~/.bashrc
```

3、创建虚拟环境
```bash
# mkvirtualenv python_start
created virtual environment CPython3.13.0.final.0-64 in 153ms
  creator CPython3Posix(dest=/root/.virtualenvs/python_start, clear=False, no_vcs_ignore=False, global=False)
  seeder FromAppData(download=False, pip=bundle, via=copy, app_data_dir=/root/.local/share/virtualenv)
    added seed packages: pip==24.3.1
  activators BashActivator,CShellActivator,FishActivator,NushellActivator,PowerShellActivator,PythonActivator
virtualenvwrapper.user_scripts creating /root/.virtualenvs/python_start/bin/predeactivate
virtualenvwrapper.user_scripts creating /root/.virtualenvs/python_start/bin/postdeactivate
virtualenvwrapper.user_scripts creating /root/.virtualenvs/python_start/bin/preactivate
virtualenvwrapper.user_scripts creating /root/.virtualenvs/python_start/bin/postactivate
virtualenvwrapper.user_scripts creating /root/.virtualenvs/python_start/bin/get_env_details
(python_start) [root@chaos-3 Python-3.13.0]# python
Python 3.13.0 (main, Nov  4 2024, 04:09:22) [GCC 8.5.0 20210514 (Red Hat 8.5.0-4)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>>
```

4、虚拟环境目录
```bash
(python_start) [root@chaos-3 opt]# ls ~/.virtualenvs/
get_env_details  postactivate	 postmkproject	   postrmvirtualenv  predeactivate  premkvirtualenv  python_start
initialize	 postdeactivate  postmkvirtualenv  preactivate	     premkproject   prermvirtualenv
```

其中 `python_start` 就是虚拟环境的地址。

5、虚拟环境常用命令
```bash
# 创建环境名
mkvirtualenv 环境名
# 退出环境
deactivate
# 进入环境
workon 环境名
# 删除环境
rmvirtualenv 环境名
# 列出所有环境
lsvirtualenv
```


### 2.3 MacOS

#### virtualenv

1、安装。如果下载比较慢，可以使用镜像
```bash
pip3 install virtualenv -i https://pypi.tuna.tsinghua.edu.cn/simple --trusted-host pypi.tuna.tsinghua.edu.cn
pip3 install virtualenvwrapper -i https://pypi.tuna.tsinghua.edu.cn/simple --trusted-host pypi.tuna.tsinghua.edu.cn
```

2、配置

找到 `virtualenvwrapper.sh` 文件。这里不同的 Linux 系统 `virtualenvwrapper.sh` 路径可能不一致，最好通过 `find` 命令查询一下。
```bash
➜  which python3
/Library/Frameworks/Python.framework/Versions/3.13/bin/python3
➜  which virtualenvwrapper.sh
/Library/Frameworks/Python.framework/Versions/3.13/bin/virtualenvwrapper.sh
```

编辑 `.zshrc` 文件：  
```bash
vim ~/.zshrc
```

在最后添加下面内容：
```bash
export VIRTUALENVWRAPPER_PYTHON=/Library/Frameworks/Python.framework/Versions/3.13/bin/python3
export WORKON_HOME=$HOME/.virtualenvs
source /Library/Frameworks/Python.framework/Versions/3.13/bin/virtualenvwrapper.sh
```

最后执行下面命令重新加载：
```bash
➜  source ~/.zshrc
virtualenvwrapper.user_scripts creating /Users/finnley/.virtualenvs/premkproject
virtualenvwrapper.user_scripts creating /Users/finnley/.virtualenvs/postmkproject
virtualenvwrapper.user_scripts creating /Users/finnley/.virtualenvs/initialize
virtualenvwrapper.user_scripts creating /Users/finnley/.virtualenvs/premkvirtualenv
virtualenvwrapper.user_scripts creating /Users/finnley/.virtualenvs/postmkvirtualenv
virtualenvwrapper.user_scripts creating /Users/finnley/.virtualenvs/prermvirtualenv
virtualenvwrapper.user_scripts creating /Users/finnley/.virtualenvs/postrmvirtualenv
virtualenvwrapper.user_scripts creating /Users/finnley/.virtualenvs/predeactivate
virtualenvwrapper.user_scripts creating /Users/finnley/.virtualenvs/postdeactivate
virtualenvwrapper.user_scripts creating /Users/finnley/.virtualenvs/preactivate
virtualenvwrapper.user_scripts creating /Users/finnley/.virtualenvs/postactivate
virtualenvwrapper.user_scripts creating /Users/finnley/.virtualenvs/get_env_details
```

3、创建虚拟环境
```bash
➜  mkvirtualenv python_learning
created virtual environment CPython3.13.0.final.0-64 in 275ms
  creator CPython3Posix(dest=/Users/finnley/.virtualenvs/python_learning, clear=False, no_vcs_ignore=False, global=False)
  seeder FromAppData(download=False, pip=bundle, via=copy, app_data_dir=/Users/finnley/Library/Application Support/virtualenv)
    added seed packages: pip==24.3.1
  activators BashActivator,CShellActivator,FishActivator,NushellActivator,PowerShellActivator,PythonActivator
virtualenvwrapper.user_scripts creating /Users/finnley/.virtualenvs/python_learning/bin/predeactivate
virtualenvwrapper.user_scripts creating /Users/finnley/.virtualenvs/python_learning/bin/postdeactivate
virtualenvwrapper.user_scripts creating /Users/finnley/.virtualenvs/python_learning/bin/preactivate
virtualenvwrapper.user_scripts creating /Users/finnley/.virtualenvs/python_learning/bin/postactivate
virtualenvwrapper.user_scripts creating /Users/finnley/.virtualenvs/python_learning/bin/get_env_details
```

列出虚拟环境列表
```
➜  lsvirtualenv
python_learning
===============


➜  workon
python_learning
```

启动/切换虚拟环境

```
workon [虚拟环境名称]
```

删除虚拟环境

```
rmvirtualenv [虚拟环境名称]
```

离开虚拟环境，和virtualenv 一样
```
deactivate
```

#### venv

1、创建虚拟环境。Python 虚拟环境用于将软件包安装与系统隔离开来。

创建一个新的虚拟环境，方法是选择 Python 解释器并创建一个 `./venv` 目录来存放它：

```bash
python3 -m venv --system-site-packages ./venv
```

Windows:
```bash
python -m venv --system-site-packages .\venv
```

使用特定于 shell 的命令激活该虚拟环境：

```bash
source ./venv/bin/activate  # sh, bash, or zsh
```

```bash
source ./venv/bin/activate.csh  # csh or tcsh
```

```bash
source ./venv/bin/activate.csh  # csh or tcsh
```

当虚拟环境处于有效状态时，shell 提示符带有 (venv) 前缀。
在不影响主机系统设置的情况下，在虚拟环境中安装软件包。首先升级 pip：

```bash
pip install --upgrade pip
```

```bash
pip list  # show packages installed within the virtual environment
```

之后退出虚拟环境：

```bash
deactivate  # don't exit until you're done using TensorFlow
```

我的操作，比如： 

```bash
Envs python3 -m venv --system-site-packages ./cat-srvs
➜  Envs source ./cat-srvs/bin/activate
(cat-srvs) ➜  Envs pip list
Package          Version
---------------- ---------
certifi          2020.12.5
chardet          4.0.0
idna             2.10
jsonrpclib-pelix 0.4.2
pip              21.0.1
requests         2.25.1
setuptools       49.2.1
urllib3          1.26.3
WARNING: You are using pip version 21.0.1; however, version 21.1.3 is available.
You should consider upgrading via the '/Library/Frameworks/Python.framework/Versions/3.9/bin/python3.9 -m pip install --upgrade pip' command.
(cat-srvs) ➜  Envs pip install --upgrade pip
Requirement already satisfied: pip in /Library/Frameworks/Python.framework/Versions/3.9/lib/python3.9/site-packages (21.0.1)
Collecting pip
  Downloading pip-21.1.3-py3-none-any.whl (1.5 MB)
     |████████████████████████████████| 1.5 MB 349 kB/s
Installing collected packages: pip
  Attempting uninstall: pip
    Found existing installation: pip 21.0.1
    Uninstalling pip-21.0.1:
      Successfully uninstalled pip-21.0.1
Successfully installed pip-21.1.3
(cat-srvs) ➜  Envs pip list
Package          Version
---------------- ---------
certifi          2020.12.5
chardet          4.0.0
idna             2.10
jsonrpclib-pelix 0.4.2
pip              21.1.3
requests         2.25.1
setuptools       49.2.1
urllib3          1.26.3
(cat-srvs) ➜  Envs deactivate
➜  Envs
```

## 更新日志

- 2024.11.04 更新 Python 版本为 3.13
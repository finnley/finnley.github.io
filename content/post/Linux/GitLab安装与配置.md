+++
title = 'GitLab安装与配置'
date = 2020-08-30T12:29:22+08:00
draft = true
categories = [ "Linux" ]
+++


# 搭建 GitLab Server

# 准备

环境： CentOS

# 关闭防火墙
关闭系统防火墙:
```shell
sudo systemctl stop firewalld
```

禁止系统防火墙开机启动：
```shell
sudo systemctl disable firewalld
```

# 关闭 SELINUX

关闭 `SELINUX` 强制访问控制安全策略以保证该策略不会影响 GitLab 正常运行：
```
sudo vim /etc/sysconfig/selinux
```

找到 `SELINUX=enforcing` 一行修改为 `SELINUX=disabled`，然后保存退出，重启系统使得 `SELINUX` 禁用生效，重启后重新登录系统，以便后续操作
可以通过 `getenforce` 命令查看 `SELINUX` 策略是否生效被禁用：
```bash
$ getenforce
Disabled
```

# 安装

文档地址: [GitLab](https://docs.gitlab.com/)

1.GitLab 依赖包
```bash
sudo yum install -y curl policycoreutils openssh-server openssh-clients postfixs
```

2.启动 postfixs 邮件服务并设置开机自启动

```bash
sudo systemctl start postfix
sudo systemctl enable postfix
```

3.使用 curl 下载 gitlab yum 仓库源

**如果在国内的话，可以尝试使用清华大学的源**

新建 `/etc/yum.repos.d/gitlab-ce.repo`，内容为:

```
[gitlab-ce]
name=Gitlab CE Repository
baseurl=https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el$releasever/
gpgcheck=0
enabled=1
```

参考：[Gitlab Community Edition 镜像使用帮助](https://mirrors.tuna.tsinghua.edu.cn/help/gitlab-ce/)

**如果在国外的话，可以使用**

```
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.rpm.sh | sudo bash
```
或者

```bash
curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash
```

4.安装 gitlab-ce 社区版 yum 一键安装包 

选择上面任意一种方式后安装 gitlab-ce

```
sudo yum makecache
sudo yum install -y gitlab-ce
```

# 使用openssl创建本地证书并配置config加载证书

1.创建ssl目录

```
sudo mkdir -p /etc/gitlab/ssl
```

2.创建本地私有秘钥

```
sudo openssl genrsa -out "/etc/gitlab/ssl/gitlab.example.com.key" 2048
```

3.使用私有秘钥创建 csr 证书

```
cd /etc/gitlab/ssl
sudo openssl req -new -key "/etc/gitlab/ssl/gitlab.example.com.key" -out "/etc/gitlab/ssl/gitlab.example.com.csr"
```

过程如下：
```
$ cd /etc/gitlab/ssl/
$ sudo openssl req -new -key "/etc/gitlab/ssl/gitlab.example.com.key" -out "/etc/gitlab/ssl/gitlab.example.com.csr"
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:cn
State or Province Name (full name) []:shh
Locality Name (eg, city) [Default City]:shh
Organization Name (eg, company) [Default Company Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (eg, your name or your server's hostname) []:gitlab.example.com
Email Address []:admin@example.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:123456
An optional company name []:
$
```

查看当前目录是否创建成功私有秘钥和csr证书
```
[root@devops-gitlab ssl]# ll
total 8
-rw-r--r--. 1 root root 1078 Oct  7 11:19 gitlab.example.com.csr
-rw-r--r--. 1 root root 1675 Oct  7 11:14 gitlab.example.com.key
[root@devops-gitlab ssl]#
```

可以看到私有秘钥和csr证书已经成功创建到本地目录

4.使用csr证书和私有秘钥创建 `crt` 天使证书

```
sudo openssl x509 -req -days 365 -in "/etc/gitlab/ssl/gitlab.example.com.csr" -signkey "/etc/gitlab/ssl/gitlab.example.com.key" -out "/etc/gitlab/ssl/gitlab.example.com.crt"
```

**x509 -req**: 表示天使证书格式
**-days 365**: 证书有效时间
**-in "/etc/gitlab/ssl/gitlab.example.com.csr"**: 引入csr证书
**-signkey "/etc/gitlab/ssl/gitlab.example.com.key"**: 引入私有密码
**-out "/etc/gitlab/ssl/gitlab.example.com.crt"**: 输入证书到指定路径

然后可以查看本地目录：

```
[vagrant@centos-gitlab ssl]$ sudo openssl x509 -req -days 365 -in "/etc/gitlab/ssl/gitlab.example.com.csr" -signkey "/etc/gitlab/ssl/gitlab.example.com.key" -out "/etc/gitlab/ssl/gitlab.example.com.crt"
Signature ok
subject=/C=cn/ST=shh/L=shh/O=Default Company Ltd/CN=gitlab.example.com/emailAddress=admin@example.com
Getting Private key

$ ll
total 12
-rw-r--r-- 1 root root 1289 Apr 30 10:34 gitlab.example.com.crt
-rw-r--r-- 1 root root 1078 Apr 30 10:28 gitlab.example.com.csr
-rw-r--r-- 1 root root 1679 Apr 30 10:21 gitlab.example.com.key
$
```

5.使用openssl创建 `pem` 证书

```
sudo openssl dhparam -out /etc/gitlab/ssl/dhparams.pem 2048
```

过程如下：

```
[vagrant@centos-gitlab ssl]$ sudo openssl dhparam -out /etc/gitlab/ssl/dhparams.pem 2048
Generating DH parameters, 2048 bit long safe prime, generator 2
This is going to take a long time
........+.............+...................................................+.......................+..+..........................+.......................................................................................................+.........................................................................................................+...............................+...............+.....................................................................................................................................................................................................................................................................................................................+...........................................................................................................................................................................................+...........................................................................................................................................................++*++*
$ ll
total 16
-rw-r--r-- 1 root root  424 Apr 30 10:36 dhparams.pem
-rw-r--r-- 1 root root 1289 Apr 30 10:34 gitlab.example.com.crt
-rw-r--r-- 1 root root 1078 Apr 30 10:28 gitlab.example.com.csr
-rw-r--r-- 1 root root 1679 Apr 30 10:21 gitlab.example.com.key
$
```

从上面可以看到pem文件成功创建，到此证书全部创建完成

6.更改当前目录 `/etc/gitlab/ssl` 下的所有证书权限

```
sudo chmod 600 *
```

```
[vagrant@centos-gitlab ssl]$ sudo chmod 600 *
[vagrant@centos-gitlab ssl]$ ll
total 16
-rw------- 1 root root  424 Apr 30 10:36 dhparams.pem
-rw------- 1 root root 1289 Apr 30 10:34 gitlab.example.com.crt
-rw------- 1 root root 1078 Apr 30 10:28 gitlab.example.com.csr
-rw------- 1 root root 1679 Apr 30 10:21 gitlab.example.com.key
[vagrant@centos-gitlab ssl]$
```

7.修改gitlab配置文件，将所有证书配置到gitlab配置文件中

```
sudo vim /etc/gitlab/gitlab.rb
```

找到下面这一行，将 `http` 改为 `https`：
```
# external_url 'http://gitlab.example.com'
修改为
external_url 'https://gitlab.example.com'
```

搜索 `redirect_http_to_https`，去掉注释，并将值设置为 `true`：
```
# nginx['redirect_http_to_https'] = false
修改为
nginx['redirect_http_to_https'] = true
```

找到下面一行：
```
# nginx['ssl_certificate'] = "/etc/gitlab/ssl/#{node['fqdn']}.crt"
修改为
# nginx['ssl_certificate'] = "/etc/gitlab/ssl/gitlab.example.com.crt"
```

找到这一行 
```
# nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/#{node['fqdn']}.key"
修改为 
# nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/gitlab.example.com.key"
``` 

找到这一样 
```
# nginx['ssl_dhparam'] = nil # Path to dhparams.pem, eg. /etc/gitlab/ssl/dhparams.pem
修改为 
# nginx['ssl_dhparam'] = /etc/gitlab/ssl/dhparams.pem # Path to dhparams.pem, eg. /etc/gitlab/ssl/dhparams.pem`
``` 

然后保存 `:wq` 保存退出

8.初始化gitlab所有服务相关配置

```
sudo gitlab-ctl reconfigure
```

9.找到gitlab下的nginx代理工具更改gitlab的http配置文件

```
sudo vim /var/opt/gitlab/nginx/conf/gitlab-http.conf
```

找到 `server_name` 这一行，在这一行的下面添加下面一句话，用来重定向所有gitlab的https请求：
```
rewrite ^(.*)$ https://$host$1 permanent;
```

保存 `:wq` 退出

10.重启gitlab使nginx配置生效

```
sudo gitlab-ctl restart
```

11.修改本地 `host` 文件，添加一条host记录

将 `gitlab.example.com` 重定向到 `192.168.16.20` 的 gitlab 主机，这样本机就可以直接使用这个域名去访问gitlab:
```
192.168.16.20  gitlab.example.com
```

到这里就成功在本地安装了gitlab分布式代码仓库

![](images/gitlab/1.png)

12.登录

第一次登录，用户名默认是 `root`， 密码可以在 `/etc/gitlab/initial_root_password` 中找到：
```
[vagrant@centos-gitlab ssl]$ sudo cat /etc/gitlab/initial_root_password
# WARNING: This value is valid only in the following conditions
#          1. If provided manually (either via `GITLAB_ROOT_PASSWORD` environment variable or via `gitlab_rails['initial_root_password']` setting in `gitlab.rb`, it was provided before database was seeded for the first time (usually, the first reconfigure run).
#          2. Password hasn't been changed manually, either via UI or via command line.
#
#          If the password shown here doesn't work, you must reset the admin password following https://docs.gitlab.com/ee/security/reset_user_password.html#reset-your-root-password.

Password: 7DYHvQacduNVuZOzoYzn/fXEs5cTU7miJ8OOTT7ln8g=

# NOTE: This file will be automatically deleted in the first reconfigure run after 24 hours.
[vagrant@centos-gitlab ssl]$
```

进入登录页面
![](images/gitlab/2.png)

13.修改管理员密码

我本地修改成了 `123123123`，修改完后会自动退出重新登录，用户默认是 `root`，密码修改后的密码 `123123123`，然后点击 `Sign in` 进入到首页



```
sudo systemctl enable sshd
sudo systemctl start sshd 
```



# GitLab 使用

1.点击 `New project` 创建一个新的代码仓库，然后选择第一个 `Create blank project`

![](images/gitlab/3.png)

![](images/gitlab/4.png)


2.创建名为 `test-repo` 的项目，然后点击 `Create project` 按钮

![](images/gitlab/5.png)
![](images/gitlab/6.png)

3.复制仓库地址 `https://gitlab.example.com/gitlab-instance-26791fe8/test-repo.git` 并克隆到本地

为了避免本地证书无法克隆操作，需要使用 `-c http.sslVerify=false` 参数

```
$ mkdir repo
$ cd repo
$ git -c http.sslVerify=false clone https://gitlab.example.com/gitlab-instance-26791fe8/test-repo.git
Cloning into 'test-repo'...
Username for 'https://gitlab.example.com': root
Password for 'https://root@gitlab.example.com':
remote: Enumerating objects: 3, done.
remote: Counting objects: 100% (3/3), done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (3/3), done.
```

如果无法clone,可以尝试使用上传本地ssh公钥的方式，再 git clone

4.进入仓库，创建测试代码

```
$ ls
test-repo
$ cd test-repo
$ sudo vim test.py
$ cat test.py
print "This is a test code"
```

6.提交文件到gitlab仓库

```
$ git config user.name "admin"
$ git config user.email "admin@example.com"
$ git status
On branch main
Your branch is up to date with 'origin/main'.

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	test.py

nothing added to commit but untracked files present (use "git add" to track)
$ git add test.py
$ git commit -m "First Commit"
[main 4d65f67] First Commit
 1 file changed, 1 insertion(+)
 create mode 100644 test.py
$ git -c http.sslVerify=false push origin main
Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Delta compression using up to 16 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 295 bytes | 295.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To https://gitlab.example.com/gitlab-instance-26791fe8/test-repo.git
   f303179..4d65f67  main -> main
```

![](images/gitlab/7.png)


## 安装 Python3

1. 下载 python3

```
wget https://www.python.org/ftp/python/3.7.9/Python-3.7.9.tar.xz
```

2. 解压

```
tar xf Python-3.7.9.tar.xz
```

3. 安装

```
cd Python-3.7.9/

yum install -y glibc glibc-devel glibc-headers glibc-common gcc

./configure --prefix=/usr/local --with-ensurepip=install --enable-shared LDFLAGS="-Wl,-rpath /usr/local/lib"

make && make altinstall
```

如果安装完提示下面信息

```
WARNING: The script easy_install-3.7 is installed in '/usr/local/bin' which is not on PATH.
  Consider adding this directory to PATH or, if you prefer to suppress this warning, use --no-warn-script-location.
  WARNING: The script pip3.7 is installed in '/usr/local/bin' which is not on PATH.
  Consider adding this directory to PATH or, if you prefer to suppress this warning, use --no-warn-script-location.
```

```
[root@devops-ansible Python-3.7.9]# which pip3.7
/usr/bin/which: no pip3.7 in (/sbin:/bin:/usr/sbin:/usr/bin)
```

解决方法

```
[root@devops-ansible Python-3.7.9]# export PATH=/usr/local/bin:$PATH
[root@devops-ansible Python-3.7.9]# which pip3.7
/usr/local/bin/pip3.7
[root@devops-ansible Python-3.7.9]#
```

4. 创建软链接

```
ln -s /usr/local/bin/pip3.7 /usr/local/bin/pip
```

5. 安装 virtualenv

```
pip install -i http://pypi.douban.com/simple/ --trusted-host pypi.douban.com virtualenv
```

## 创建 Ansible 系统账户

1. 创建 deploy 用户

```
useradd deploy
```

2. 登录到 deploy 的命令行界面

```
su - deploy
```

3. 在deploy用户下创建env实例

```
virtualenv -p /usr/local/bin/python3.7 .py3-a2.10-env
```

```
[deploy@devops-ansible ~]$ virtualenv -p /usr/local/bin/python3.7 .py3-a2.10-env
created virtual environment CPython3.7.9.final.0-64 in 896ms
  creator CPython3Posix(dest=/home/deploy/.py3-a2.10-env, clear=False, global=False)
  seeder FromAppData(download=False, pip=bundle, setuptools=bundle, wheel=bundle, via=copy, app_data_dir=/home/deploy/.local/share/virtualenv)
    added seed packages: pip==20.2.3, setuptools==50.3.0, wheel==0.35.1
  activators BashActivator,CShellActivator,FishActivator,PowerShellActivator,PythonActivator,XonshActivator
[deploy@devops-ansible ~]$
```

4. 安装 ansiable

```
cd /home/deploy/.py3-a2.10-env/

which git

git clone https://github.com/ansible/ansible.git

source /home/deploy/.py3-a2.10-env/bin/activate
```

```
[deploy@devops-ansible ~]$ cd /home/deploy/.py3-a2.10-env/
[deploy@devops-ansible .py3-a2.10-env]$ which git
/bin/git
[deploy@devops-ansible .py3-a2.10-env]$ git clone https://github.com/ansible/ansible.git
Cloning into 'ansible'...
remote: Enumerating objects: 539719, done.
remote: Counting objects: 100% (539719/539719), done.
remote: Compressing objects: 100% (128106/128106), done.
remote: Total 539719 (delta 361943), reused 539719 (delta 361943), pack-reused 0
Receiving objects: 100% (539719/539719), 179.14 MiB | 8.72 MiB/s, done.
Resolving deltas: 100% (361943/361943), done.
[deploy@devops-ansible .py3-a2.10-env]$
[deploy@devops-ansible .py3-a2.10-env]$ source /home/deploy/.py3-a2.10-env/bin/activate
(.py3-a2.10-env) [deploy@devops-ansible .py3-a2.10-env]$
```

5. 安装依赖包

```
pip install -i http://pypi.douban.com/simple/ --trusted-host pypi.douban.com paramiko PyYAML jinja2
```

6. 移动 ansible 到虚拟环境，并进入到该目录

我好想就是在在 /home/deploy/.py3-a2.10-env/ 目录中clone的

```
cd ansible
git checkout stable-2.10
source /home/deploy/.py3-a2.10-env/ansible/hacking/env-setup -q
```

7. 验证 ansible 2.10 是否加载完成

```
ansible --version

ansible-playbook --version
```

```
(.py3-a2.10-env) [deploy@devops-ansible ansible]$ git checkout stable-2.10
Branch stable-2.10 set up to track remote branch stable-2.10 from origin.
Switched to a new branch 'stable-2.10'
(.py3-a2.10-env) [deploy@devops-ansible ansible]$ source /home/deploy/.py3-a2.10-env/ansible/hacking/env-setup -q
(.py3-a2.10-env) [deploy@devops-ansible ansible]$ ansible --version
ansible 2.10.2.post0
  config file = None
  configured module search path = ['/home/deploy/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /home/deploy/.py3-a2.10-env/ansible/lib/ansible
  executable location = /home/deploy/.py3-a2.10-env/ansible/bin/ansible
  python version = 3.7.9 (default, Oct  8 2020, 08:05:12) [GCC 4.8.5 20150623 (Red Hat 4.8.5-39)]
(.py3-a2.10-env) [deploy@devops-ansible ansible]$ ansible-playbook --version
ansible-playbook 2.10.2.post0
  config file = None
  configured module search path = ['/home/deploy/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /home/deploy/.py3-a2.10-env/ansible/lib/ansible
  executable location = /home/deploy/.py3-a2.10-env/ansible/bin/ansible-playbook
  python version = 3.7.9 (default, Oct  8 2020, 08:05:12) [GCC 4.8.5 20150623 (Red Hat 4.8.5-39)]
(.py3-a2.10-env) [deploy@devops-ansible ansible]$
```

到这里已经成功在 ansible 主机下完成了在py3.7虚拟环境下安装ansible2.10版本，确保ansible在一个独立环境下正常稳定运转

## ansible playbookds 入门

1. 创建 test_playbooks 目录,配置testservers主列表

```
(.py3-a2.10-env) [deploy@devops-ansible ~]$ mkdir test_playbooks
(.py3-a2.10-env) [deploy@devops-ansible ~]$ cd test_playbooks/
(.py3-a2.10-env) [deploy@devops-ansible test_playbooks]$ mkdir inventory
(.py3-a2.10-env) [deploy@devops-ansible test_playbooks]$ mkdir roles
(.py3-a2.10-env) [deploy@devops-ansible test_playbooks]$ cd inventory/
(.py3-a2.10-env) [deploy@devops-ansible inventory]$ vim testenv
(.py3-a2.10-env) [deploy@devops-ansible inventory]$ cat testenv
[testservers]
test.example.com

[testservers:vars]
server_name=test.example.com
user=root
output=/root/test/txt
(.py3-a2.10-env) [deploy@devops-ansible inventory]$
```

2. 添加tasks测试任务，在目标主机下输出带有参数的语句到目标主机的某个路径下

```
(.py3-a2.10-env) [deploy@devops-ansible inventory]$ cd ../roles/
(.py3-a2.10-env) [deploy@devops-ansible roles]$ mkdir testbox
(.py3-a2.10-env) [deploy@devops-ansible roles]$ cd testbox/
(.py3-a2.10-env) [deploy@devops-ansible testbox]$ mkdir tasks
(.py3-a2.10-env) [deploy@devops-ansible testbox]$ cd tasks/
(.py3-a2.10-env) [deploy@devops-ansible tasks]$ vim main.yml
(.py3-a2.10-env) [deploy@devops-ansible tasks]$ cat main.yml
- name: print server name and user to remove testbox
  shell: "echo 'Currently {{ user }} is logining {{ server_name }}' > {{ output }}"
(.py3-a2.10-env) [deploy@devops-ansible tasks]$
```

3. 创建任务入口文件

```
(.py3-a2.10-env) [deploy@devops-ansible tasks]$ cd ../../..
(.py3-a2.10-env) [deploy@devops-ansible test_playbooks]$ pwd
/home/deploy/.py3-a2.10-env/ansible/test_playbooks
(.py3-a2.10-env) [deploy@devops-ansible test_playbooks]$ vim deploy.yml
(.py3-a2.10-env) [deploy@devops-ansible test_playbooks]$ cat deploy.yml
- hosts: "testservers"
  gather_facts: true
  remote_user: root
  roles:
    - testbox
(.py3-a2.10-env) [deploy@devops-ansible test_playbooks]$
(.py3-a2.10-env) [deploy@devops-ansible test_playbooks]$ tree .
.
├── deploy.yml
├── inventory
│   └── testenv
└── roles
    └── testbox
        └── tasks
            └── main.yml

4 directories, 3 files
(.py3-a2.10-env) [deploy@devops-ansible test_playbooks]$
```

## 配置 Ansible playbooks SSH 免密码秘钥认证

1. 使用 root 身份修改 hosts 添加一条记录

```
su - root
[root@devops-ansible ~]# vim /etc/hosts
[root@devops-ansible ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
127.0.1.1 devops-ansible devops-ansible

192.168.205.121 test.example.com
[root@devops-ansible ~]#
```

```
192.168.205.120 test.example.com
```

2. 返回 deploy 目录

```
su - deploy
[deploy@devops-ansible ~]$ source /home/deploy/.py3-a2.10-env/bin/activate
(.py3-a2.10-env) [deploy@devops-ansible ~]$
```

在使用 ansible 执行 playbooks 之前，因为 ansible 是以 ssh 作为通讯协议，为了保证正常的通信连接，需要配置ansible主机与目标主机的秘钥认证，保证ansible无需密码就可以访问目标主机

3. ansible 服务器端创建爱你ssh本地秘钥

```
ssh-keygen -t rsa
```

```
(.py3-a2.10-env) [deploy@devops-ansible ~]$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/deploy/.ssh/id_rsa):
Created directory '/home/deploy/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/deploy/.ssh/id_rsa.
Your public key has been saved in /home/deploy/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:96nWzhxy44CQCHyr6Ak3MxBZwxBC4P9muWWImpHUsQQ deploy@devops-ansible
The key's randomart image is:
+---[RSA 2048]----+
|BEo              |
|o+o.             |
|o.oo.            |
| .+ooo .         |
|.. +o o S .      |
|.o..o o. o . .   |
|oo*. * o. o.*    |
|o.=+o +   .O.o   |
| =   .   ...=    |
+----[SHA256]-----+
(.py3-a2.10-env) [deploy@devops-ansible ~]$
```

2. ansible 服务器端建立与目标部署及其的秘钥认证

```
ssh-copy-id -i /home/deploy/.ssh/id_rsa.pub root@test.example.com
```

```
(.py3-a2.10-env) [deploy@devops-ansible ~]$ ssh-copy-id -i /home/deploy/.ssh/id_rsa.pub root@test.example.com
/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/deploy/.ssh/id_rsa.pub"
/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@test.example.com's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'root@test.example.com'"
and check to make sure that only the key(s) you wanted were added.

(.py3-a2.10-env) [deploy@devops-ansible ~]$ ssh root@test.example.com
Last login: Thu Oct  8 16:21:59 2020
[root@devops-gitlab ~]#
```

上面方法可能连接不上，需要登录另外一个服务器进行设置

https://www.appblog.cn/2019/12/04/%E8%A7%A3%E5%86%B3Vagrant%E4%B8%AD%E7%9A%84CentOS%E4%B8%BB%E6%9C%BA%E6%97%A0%E6%B3%95ssh%E8%BF%9C%E7%A8%8B%E8%BF%9E%E6%8E%A5%E7%9A%84%E9%97%AE%E9%A2%98/

此时不需要密码就可以在 anbible 机器上登录另外一台机器

3. 执行 playbookds，部署到 testenv 环境

```
(.py3-a2.10-env) [deploy@devops-ansible ~]$ pwd
/home/deploy
(.py3-a2.10-env) [deploy@devops-ansible ~]$ ls
test_playbooks
(.py3-a2.10-env) [deploy@devops-ansible ~]$ cd test_playbooks/
(.py3-a2.10-env) [deploy@devops-ansible test_playbooks]$ ls
deploy.yml  inventory  roles
(.py3-a2.10-env) [deploy@devops-ansible test_playbooks]$ ansible-playbook -i inventory/testenv ./deploy.yml

PLAY [testservers] ************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************
ok: [test.example.com]

TASK [print server name and user to remove testbox] ***************************************************************
changed: [test.example.com]

PLAY RECAP ********************************************************************************************************
test.example.com           : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

(.py3-a2.10-env) [deploy@devops-ansible test_playbooks]$
```

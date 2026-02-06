+++
title = 'Ubuntu Linux 服务器安全加固手册'
date = 2026-02-06T09:02:45+08:00
draft = true
categories = [ "运维" ]
tags = [ "linux", "ubuntu" ]
+++

## 1. 账户安全与权限管理 (User & Privileges)

重装系统后的第一件事不是安装软件，而是收回系统的控制权。

### 1.1 创建新管理员账户

不要使用默认的 `ubuntu` 或 `root` 账户进行日常操作。

```bash
# 1. 创建用户 jack（替换你自己的实际账户）
sudo adduser jack

# 2. 将 jack 加入 sudo 组（赋予管理员权限）
# 系统会提示你设置密码，请设置一个强密码（用于sudo提权，而不是远程登录）
# 之后的一路回车即可
sudo usermod -aG sudo jack


# 3. 切换到新用户进行后续操作
su - jack
```

### 1.2 将 SSH 密钥权限复制给 jack

这样你就可以用同一个密钥登录 `jack`，而不需要密码。
```bash
# 创建 .ssh 目录
sudo mkdir -p /home/jack/.ssh

# 复制 ubuntu 用户的公钥到 einscat 用户下
sudo cp /home/ubuntu/.ssh/authorized_keys /home/jack/.ssh/

# 修正权限（将文件所有者改为 jack
sudo chown -R jack:jack /home/jack/.ssh

# 赋予正确的读写权限
sudo chmod 700 /home/jack/.ssh
sudo chmod 600 /home/jack/.ssh/authorized_keys
```

### 1.3 测试登录

**不要关闭当前窗口！** 打开一个新的终端窗口，尝试用 `jack` 登录：
```bash
ssh -i [你的密钥路径] jack@[服务器IP]
```

### 1.3 强制 SSH 安全策略 (关键)

这是黑客入侵最主要的途径。我们将禁用密码登录，仅允许密钥登录，并修改端口。

**第一步：配置 SSH 密钥 (在你的本地电脑上操作)**
如果你还没有密钥对，请生成：

```bash
# 本地生成密钥 (如果是 Windows 使用 PowerShell 或 CMD)
ssh-keygen -t ed25519 -C "your_email@example.com"

```

将公钥上传到服务器：

```bash
# 将公钥发送到服务器的 einscat 账户下
ssh-copy-id -i ~/.ssh/id_ed25519.pub einscat@<服务器IP>

```

**第二步：修改 SSH 配置文件 (在服务器上操作)**
编辑配置文件：

```bash
sudo vim /etc/ssh/sshd_config
```

修改或添加以下参数（请仔细核对）：

```ssh
# 修改默认端口 (选一个 1024-65535 之间的非常用端口，例如 22999)
Port 22999
我的：4433

# 禁止 root 用户远程登录
PermitRootLogin no

# 禁止使用密码登录 (仅允许密钥)
PasswordAuthentication no

# 仅允许 einscat 和 ubuntu 用户登录 (可选，更严格)
AllowUsers einscat ubuntu
```

**第三步：重启 SSH 服务**

> **⚠️ 警告**：在断开当前连接前，请务必新开一个终端窗口，尝试用新端口和密钥登录测试，确保配置无误，否则你将无法连接服务器！

```bash
sudo systemctl restart sshd
```

## 2. 网络防火墙 (Network Firewall)

仅仅依靠云厂商的安全组是不够的，服务器内部必须开启防火墙。Ubuntu 自带的 UFW 非常好用。我们要通过“白名单”模式（默认拒绝一切，只允许特定端口）来防御。

### 2.1 配置 UFW (Uncomplicated Firewall)


```bash
# 1. 设置默认策略：拒绝所有进入，允许所有流出
sudo ufw default deny incoming
sudo ufw default allow outgoing

# 2. 放行你刚才修改的 SSH 端口 (例如 22999)
sudo ufw allow 22999/tcp
sudo ufw allow 4433/tcp
我的：4433

# 3. 放行 Web 服务端口 (如果你需要建站)
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# 4. 启用防火墙
sudo ufw enable

```

检查状态：

```bash
sudo ufw status verbose

```

完整内容：
```
finnley@VM-0-2-ubuntu:~$ sudo ufw default deny incoming
sudo ufw default allow outgoing
Default incoming policy changed to 'deny'
(be sure to update your rules accordingly)
Default outgoing policy changed to 'allow'
(be sure to update your rules accordingly)
finnley@VM-0-2-ubuntu:~$ sudo ufw allow 4433/tcp
Rules updated
Rules updated (v6)
finnley@VM-0-2-ubuntu:~$ sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
Rules updated
Rules updated (v6)
Rules updated
Rules updated (v6)
finnley@VM-0-2-ubuntu:~$ sudo ufw enable
Firewall is active and enabled on system startup
finnley@VM-0-2-ubuntu:~$ sudo ufw status verbose
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
4433/tcp                   ALLOW IN    Anywhere                  
80/tcp                     ALLOW IN    Anywhere                  
443/tcp                    ALLOW IN    Anywhere                  
4433/tcp (v6)              ALLOW IN    Anywhere (v6)             
80/tcp (v6)                ALLOW IN    Anywhere (v6)             
443/tcp (v6)               ALLOW IN    Anywhere (v6)             

```


---

## 3. 安装 Fail2Ban（防暴破神器）

即使你禁用了密码登录，Fail2Ban 仍然有用，它可以自动封禁那些不断尝试连接你 SSH 端口的恶意 IP。Fail2Ban 会监控日志，自动封禁多次试错的 IP 地址。

### 3.1 安装与配置

```bash
sudo apt update
sudo apt install fail2ban -y

```

### 3.2 配置规则

创建一个本地配置文件（不要直接改 jail.conf）：

```bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo vim /etc/fail2ban/jail.local

```

找到 `[sshd]` 部分，确保启用并指向正确的端口：

```ini
[sshd]
enabled = true
port    = 22999
logpath = %(sshd_log)s
backend = %(sshd_backend)s
maxretry = 3
bantime = 1d
```

- 向下滚动找到 `[sshd]` 部分，确保 `enabled = true`。
- 你可以修改 `bantime`（封禁时间），例如改为 `1d` (一天)。

重启服务：

```bash
sudo systemctl restart fail2ban
```


## 4. 系统完整性与自动更新（可选）

病毒往往利用已知漏洞入侵。让系统自动打补丁是最省心的办法。

### 4.1 开启自动安全更新

不要让人为的懒惰成为漏洞。

```bash
# 安装工具
sudo apt install unattended-upgrades -y
# 启用自动更新配置向导，在弹出的界面中选择 **Yes**，系统就会自动每天检查并安装安全更新。
sudo dpkg-reconfigure -plow unattended-upgrades

```

### 4.2 锁定关键文件 (高级)

为了防止类似之前的 Rootkit 修改系统预加载文件，我们可以锁定关键文件。

```bash
# 锁定 ld.so.preload (防止 Rootkit 劫持)
sudo touch /etc/ld.so.preload
sudo chattr +i /etc/ld.so.preload

# 如果需要修改该文件，需使用 chattr -i 解锁

```

## 5. 历史命令记录优化

黑客入侵后通常会清除 history，我们需要加强日志记录。

编辑 `~/.bashrc`：

```bash
vim ~/.bashrc

```

添加以下内容：

```bash
# 记录命令执行的时间
export HISTTIMEFORMAT="%F %T "
# 增加历史记录容量
export HISTSIZE=10000
export HISTFILESIZE=20000
# 防止重复记录和空格开头的命令记录（保护隐私）
export HISTCONTROL=ignoreboth

```

使配置生效：`source ~/.bashrc`

## 6. 日常检查清单 (Checklist)

如果感觉服务器异常，执行以下检查：

1. **检查端口**：`sudo ss -tulpn` (查看是否有未知端口开放)
2. **检查进程**：`top` 或 `htop` (查看 CPU 占用)
3. **检查登录日志**：`sudo last` (查看近期谁登录过)
4. **检查 Rootkit**：建议安装 `rkhunter` 进行定期扫描。
* `sudo apt install rkhunter`
* `sudo rkhunter --check`

---

在ubuntu云服务器的下面路径已经对应的病毒：
/proc/1196/root/tmp/libsystem.so → Linux.Rootkit.Processhider.a
/proc/5038/root/tmp/libsystem.so → Linux.Rootkit.Processhider.a
/tmp/gan → Elf32.Trojan.BillGates.da
/KaBot → Elf32.Trojan.BillGates.da
/usr/bin/bsd-port/getty → Elf32.Trojan.BillGates.da
它们都是什么，该如何防范于补救。

我决定重装云服务器，系统是ubuntu，系统的默认的账户名也是“ubuntu”，安装的时候是选择输入密码还是关联秘钥，如果使用秘钥，未来更换了我本地电脑如何处理。如果重装系统完成后要求再创建账户，请取名为“einscat”。

重装系统后我该如何加固服务器安全，这些安全措施并不仅仅针对以上检查出来的病毒（这一点很重要，很重要，很重要），请提供详细的操作步骤。



这是一个非常典型的 Linux 服务器入侵案例。你所遇到的恶意软件组合（**BillGates 僵尸网络** + **Processhider Rootkit**）是云服务器最常见的攻击组合之一。

鉴于你已经重装了系统，这是一个最彻底且正确的决定。清除 Rootkit 极其困难，重装是唯一能保证 100% 干净的方法。

以下内容将分为三个部分：**威胁分析**、**基础账户设置（einscat）**、**系统加固全流程**。这篇回答采用了适合 Markdown 文档的格式，你可以直接复制保存。

---

# 🛡️ Ubuntu 服务器安全加固手册

## 第一部分：威胁分析 (本次入侵复盘)

你发现的病毒文件属于典型的 DDoS 僵尸网络套件：

1. **Trojan.BillGates (BillGates/Gates)**:
* **位置**: `/tmp/gan`, `/KaBot`, `/usr/bin/bsd-port/getty`
* **功能**: 这是一个臭名昭著的 DDoS 攻击木马。它会将你的服务器变成“肉鸡”，用来攻击其他目标。它通常会替换系统核心命令（如 `ps`, `netstat`, `lsof`）来伪装自己。
* **入侵方式**: 90% 的情况是通过 **SSH 弱密码爆破** 进入的。


2. **Rootkit.Processhider (libsystem.so)**:
* **位置**: `/proc/.../root/tmp/libsystem.so`
* **功能**: 这是一个动态链接库劫持工具。它利用 `/etc/ld.so.preload` 技术，在系统命令运行前加载自己。它的作用是**隐藏病毒进程**，让你在使用 `top` 或 `ps` 命令时看不到 BillGates 木马在运行。



---

## 第二部分：用户与权限管理

为了避免使用默认账户带来的风险，我们将创建一个新的管理员账户，并禁用默认账户。

### 1. 创建新用户 (einscat)

**操作目的**：创建名为 `einscat` 的用户，赋予 sudo 权限，并设置强密码。

```bash
# 1. 创建用户 (系统会提示你输入密码，请务必设置复杂的强密码)
sudo adduser einscat

# 2. 将新用户加入 sudo 组 (赋予管理员权限)
sudo usermod -aG sudo einscat

# 3. 验证权限 (切换到新用户测试)
su - einscat
sudo whoami
# 如果输出 "root"，则说明权限设置成功

```

### 2. 这里的关键是“密码”

在安装/创建用户时，**不要使用**生日、常用单词或纯数字。建议格式：`大写+小写+数字+符号`，长度大于 12 位。

---

## 第三部分：服务器深度加固 (核心步骤)

重装系统只是开始，如果不加固，黑客会在几分钟内再次通过全网扫描工具攻破你的服务器。

### 步骤一：配置 SSH 密钥登录 (彻底杜绝爆破)

这是**最重要**的一步。BillGates 木马主要靠猜密码进入。如果你禁用了密码登录，改用密钥，他们就无法通过这种方式入侵。

#### 1. 在你的本地电脑（不是服务器）生成密钥对

如果你是 Windows (PowerShell) 或 Mac/Linux：

```bash
ssh-keygen -t ed25519 -C "einscat_server_key"
# 一路回车即可，生成的公钥在 ~/.ssh/id_ed25519.pub

```

#### 2. 将公钥上传到服务器

在你的**本地电脑**执行：

```bash
# 将公钥发送给服务器的 einscat 用户
ssh-copy-id -i ~/.ssh/id_ed25519.pub einscat@<你的服务器IP>

```

#### 3. 测试密钥登录

尝试登录：`ssh einscat@<你的服务器IP>`。如果不需要输入密码直接进去了，说明配置成功。

### 步骤二：SSH 服务端加固

**操作目的**：修改 SSH 配置文件，禁止 root 登录，禁止密码验证。

编辑配置文件：

```bash
sudo nano /etc/ssh/sshd_config

```

找到并修改（或添加）以下参数：

```ssh
# 修改默认端口 (可选，建议修改为 10000-65535 之间的非常用端口，例如 22222)
Port 22222

# 禁止 root 直接登录 (必须)
PermitRootLogin no

# 禁止密码验证 (关键！必须在密钥登录测试成功后开启)
PasswordAuthentication no

# 禁止空密码
PermitEmptyPasswords no

# 限制仅允许 einscat 用户登录
AllowUsers einscat

```

保存并退出 (`Ctrl+O`, `Enter`, `Ctrl+X`)，然后重启 SSH 服务：

```bash
sudo systemctl restart sshd

```

*> 注意：如果修改了端口，下次登录需要加 -p 参数，如 `ssh -p 22222 einscat@IP*`

### 步骤三：启用 UFW 防火墙

**操作目的**：关闭所有非必要的端口，只留下 SSH 和 Web 服务端口。

```bash
# 1. 默认拒绝所有传入连接，允许传出
sudo ufw default deny incoming
sudo ufw default allow outgoing

# 2. 放行 SSH 端口 (如果你修改了端口，这里必须填写修改后的端口号！)
sudo ufw allow 22222/tcp  # 假设你上面改成了 22222
# 如果没改端口，则使用: sudo ufw allow ssh

# 3. 放行 Web 服务端口 (如果你跑网站的话)
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# 4. 启用防火墙
sudo ufw enable

```

### 步骤四：安装 Fail2Ban (防爆破神器)

**操作目的**：如果有人试图暴力破解你的 SSH（即使你换了端口），Fail2Ban 会在几次失败后自动封锁对方 IP。

```bash
# 1. 安装
sudo apt update
sudo apt install fail2ban -y

# 2. 创建本地配置文件
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local

# 3. 编辑配置
sudo nano /etc/fail2ban/jail.local

```

在 `jail.local` 中找到 `[sshd]` 部分，确保以下设置（如果修改了 SSH 端口，这里也要改）：

```ini
[sshd]
enabled = true
port    = 22222   # 填写你的实际 SSH 端口
logpath = %(sshd_log)s
backend = %(sshd_backend)s
maxretry = 3      # 尝试 3 次失败就封禁
bantime = 1d      # 封禁 1 天 (支持 m/h/d)

```

重启 Fail2Ban：

```bash
sudo systemctl restart fail2ban

```

### 步骤五：自动安全更新

**操作目的**：确保系统漏洞能第一时间被修复。

```bash
sudo apt install unattended-upgrades -y
sudo dpkg-reconfigure --priority=low unattended-upgrades
# 选择 "Yes"

```

---

## 第四部分：日常维护建议

1. **定期检查系统进程**：
虽然 BillGates 病毒会替换 `ps`，但加固后它很难进来。你可以安装 `htop` 来查看进程：
```bash
sudo apt install htop

```


2. **Rootkit 扫描工具**：
定期运行 Rkhunter 检查是否有异常文件。
```bash
sudo apt install rkhunter -y
sudo rkhunter --propupd # 更新文件属性数据库
sudo rkhunter --check   # 开始检查

```


3. **处理旧账户**：
当你确认 `einscat` 用户可以正常使用 sudo 且 SSH 密钥登录无误后，建议锁定默认的 `ubuntu` 账户：
```bash
sudo usermod -L ubuntu
# 或者修改其 shell 禁止登录
sudo usermod -s /usr/sbin/nologin ubuntu

```



### 总结

你之前的服务器是因为**端口开放**且**使用了可被爆破的密码**（或者默认账户未禁用）导致被批量扫描脚本攻破的。
只要你完成了**“SSH 密钥登录”**和**“禁用密码登录”**这两步，99% 的自动化攻击脚本（包括 BillGates）都将无法攻破你的服务器。


---
## 其他

没问题，我们立刻解决这个问题。

既然你要把所有项目直接放在 `/opt` 根目录下（例如 `/opt/project-a`，`/opt/project-b`），而 `/opt` 本身属于 `root`，你无法直接由 `git clone` 自动创建目录。

请**直接执行**以下 3 步标准操作（每次新项目都这样做），这是最符合规范且能解决权限问题的方案：

### 操作步骤

假设你要 Clone 的项目名字叫 **`my-app`** (请将下面的 `my-app` 替换为你实际的项目目录名)：

1. **先用管理员权限创建空目录**：
```bash
sudo mkdir -p /opt/my-app

```


2. **把这个目录送给 `einscat` (关键一步)**：
*这样以后你在这个目录里做任何事都不用再输 sudo 了。*
```bash
sudo chown -R einscat:einscat /opt/my-app

```


3. **直接 Clone 代码到该目录**：
*注意命令最后有个空格和点 `.`，表示克隆到当前目录，而不是再建一层文件夹。*
```bash
# 方式 A：指定目录 clone (推荐)
git clone <你的git地址> /opt/my-app

# 方式 B：进去后再 clone
cd /opt/my-app
git clone <你的git地址> .

```



---

### 验证

执行完后，你可以输入 `ls -ld /opt/my-app`，看到的应该是：
`drwxr-xr-x ... einscat einscat ... /opt/my-app`
(只要看到 **einscat** 就说明成功了)。

---

2026-02-06 11:18:50+08:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 5.7.44-1.el7 started.
2026-02-06 11:18:50+08:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
2026-02-06 11:18:50+08:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 5.7.44-1.el7 started.
2026-02-06 11:18:50+08:00 [Note] [Entrypoint]: Initializing database files
2026-02-06T03:18:50.896117Z 0 [ERROR] Could not open file '/var/log/mysql/error.log' for error logging: Permission denied
2026-02-06T03:18:50.896150Z 0 [ERROR] Aborting


---

这是最让人头疼的操作之一，解决办法非常标准。只要把你当前的账号加入到 `docker` 用户组，Docker 守护进程就会把你视作“自己人”，不再需要 `sudo`。

请在终端依次执行以下 3 步：

### 第一步：将当前用户加入 Docker 组

*(这一步依然需要最后一次 sudo)*

```bash
sudo usermod -aG docker einscat

```

* `-aG` 的意思是：Append (追加) 到 Group (组)。千万别漏了 `-a`，否则你会退出其他所有组。

### 第二步：立即刷新组身份

通常修改组后需要注销重新登录才能生效，但用下面这个命令可以让它**立即在当前窗口生效**：

```bash
newgrp docker

```

### 第三步：验证

现在直接运行下面命令（不加 sudo）：

```bash
docker ps

```

如果没报错，直接输出了容器列表（哪怕是空的），说明搞定！

---

### ⚠️ 重要提示

* **关于安全性**：加入 `docker` 组等同于获得了 Root 权限（因为你可以通过挂载 `/` 目录来修改系统文件）。这在开发服务器上是标准操作，但你要心里有数。
* **多窗口问题**：`newgrp` 只对**当前**终端窗口生效。如果你开了其他 SSH 窗口，那边的窗口可能还需要注销重连一次才能不用 `sudo`。下次你重新 SSH 连进来时，就自动永久生效了。


---
finnley@VM-0-2-ubuntu:~$ sudo rm -rf /persist/
finnley@VM-0-2-ubuntu:~$ sudo mkdir -p /persist/data
finnley@VM-0-2-ubuntu:~$ sudo mkdir -p /persist/logs
finnley@VM-0-2-ubuntu:~$ sudo mkdir -p /persist/logs/mysql5.7
finnley@VM-0-2-ubuntu:~$ sudo chown -R 999:999 /persist/logs/mysql5.7
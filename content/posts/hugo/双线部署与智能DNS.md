+++
title = '双线部署与智能DNS'
date = 2026-02-05T22:01:30+08:00
draft = false
categories = [ "站点" ]
tags = [ "hugo" ]
+++


## 1 目标

1. **国外访问**：走 Vercel 自动构建（带广告）。
2. **国内访问**：走 GitHub Actions 自动推送到你的 Ubuntu 服务器（无广告）。
3. **安全性**：使用专用的低权限账号 `github-deploy`，杜绝 root 权限泄露风险。

## 2 GitHub推送站点文件至服务器

为了安全起见，**绝对应该**创建一个权限受限的专用账户。使用 `root` 账户进行自动化部署是非常危险的，一旦私钥泄露，整个服务器的控制权就丢了。

以下是在 Ubuntu 系统上创建“最小权限部署用户”的完整操作指南。

### 2.1 核心思路

1. **创建用户：** 创建一个名为 `github-deploy` 的用户。
2. **配置 SSH：** 只允许该用户通过密钥登录（禁止密码登录）。
3. **配置目录权限：** 只给予该用户对“网站目录”的读写权限，不给 `sudo` 权限，无法修改系统文件。


### 2.2 第一阶段：本地准备（生成“钥匙”和“锁”）

**📍 操作地点：你的本地电脑 (Windows/Mac)**
**⚠️ 注意：不要在服务器上运行这一步，要在你平时写代码的电脑上运行。**

1. 打开你的终端（CMD, PowerShell 或 Terminal）。
2. 输入以下命令生成密钥对：
```bash
ssh-keygen -t rsa -b 4096 -C "github-deploy-key" -f id_rsa_deploy
```

* *提示输入密码时：* 直接按回车（不设置密码），否则 GitHub Actions 无法自动登录。

3. 执行完后，你当前目录下会出现两个文件：

- `id_rsa_deploy` (无后缀，私钥/钥匙)：这是**私钥**。
    - 🔑 **给谁：** 给 GitHub (填入 Secrets)。
    - ⚠️ **注意：** 就像你家门的钥匙，绝对不能给别人，**严禁泄露**，也不能发给服务器，稍后填入 GitHub。

- `id_rsa_deploy.pub` (有 `.pub` 后缀，公钥/锁芯)：这是**公钥**。
    - 🔒 **给谁：** 给服务器 (填入 `authorized_keys`)。
    - ⚠️ **注意：** 就像你家门的锁芯，你需要把它装到服务器上，这样拿钥匙（私钥）的人才能开门，稍后放入服务器。


### 2.3 第二阶段：服务器配置（安装“锁”和创建“看门人”）

**📍 操作地点：你的国内 Ubuntu 服务器 (使用 root 或 sudo 账号登录)**

1、创建专用部署用户

登录到你的服务器（使用目前的 root 或 sudo 账号），执行以下命令：

```bash
# 1. 创建用户 (名称随意，这里用 github-deploy)
sudo adduser github-deploy

# 系统会提示设置密码，随便设置一个复杂的即可（后面我们会禁止用密码登录，只用密钥）
# 其他信息（如 Full Name）直接回车跳过
```

2、安装公钥配置 SSH 免密登录（把锁装上）

我们需要把你之前生成的**公钥**（`id_rsa_deploy.pub`）放到这个新用户的“白名单”里。

```bash
# 1. 切换到新用户
sudo su - github-deploy

# 2. 创建 .ssh 目录
mkdir .ssh
chmod 700 .ssh

# 3. 创建 authorized_keys 授权文件并编辑
nano .ssh/authorized_keys
```

* **操作：** 打开你本地电脑上的 `id_rsa_deploy.pub` 文件，复制里面的所有内容，**粘贴**到这个编辑器里。
* **保存：** 按 `Ctrl + O` -> 回车 -> `Ctrl + X` 退出。

```bash
# 设置权限（关键！）
chmod 600 .ssh/authorized_keys

# 退出 github-deploy 用户，切回 root
exit
```

3、配置网站目录权限（给看门人钥匙）

假设你的网站根目录在 `/www/wwwroot/mysite`（请替换为你实际的目录）。
目前这个目录的所有者通常是 `root` 或 `www-data`，新用户 `github-deploy` 默认是无法写入的。

我们需要把这个目录的控制权交给 `github-deploy`，同时保证 Nginx（通常运行为 `www-data` 用户）依然能读取文件。

```bash
# 1. 将网站目录的所有者改为 github-deploy，组改为 www-data
# 这样 github-deploy 可以写入，Nginx (www-data) 可以读取
sudo chown -R github-deploy:www-data /www/wwwroot/mysite
sudo chown -R github-deploy:www-data /opt/hugo-site-notes

# 2. 设置目录权限：所有者(rwx) 组(rx) 其他人(rx)
# 755 是标准权限，确保 Nginx 能读取，github-deploy 能写入
sudo chmod -R 755 /www/wwwroot/mysite
```

4、(可选但推荐) 安全加固，禁用该用户密码登录

为了防止有人尝试暴力破解这个用户的密码，建议禁止这个用户通过密码登录 SSH。

在服务器上（使用 root/sudo 账号）：

```bash
sudo vim /etc/ssh/sshd_config
```

在文件末尾添加（或修改）以下配置：

```ssh
# 针对 github-deploy 用户，禁止密码登录，只允许密钥
Match User github-deploy
    PasswordAuthentication no
    PermitRootLogin no
    AllowTcpForwarding no
    X11Forwarding no

```

保存并重启 SSH 服务：

```bash
sudo service ssh restart
```

### 2.4 第三阶段：GitHub 配置（把“钥匙”交给机器人）

**📍 操作地点：GitHub 网页端**

1. 打开你的博客仓库 -> **Settings** -> **Secrets and variables** -> **Actions**。
2. 点击 **New repository secret**，添加以下三个变量：

| Secret Name | Secret Value (填什么) |
| --- | --- |
| **REMOTE_HOST** | 你的国内服务器公网 IP |
| **REMOTE_USER** | `github-deploy` |
| **SERVER_SSH_KEY** | 打开本地的 **`id_rsa_deploy` (私钥)**，复制**全部**内容粘贴进去 |


### 2.5 第四阶段：代码配置（设置“分流”逻辑）

**📍 操作地点：你的本地代码编辑器**

1、确认 `hugo.toml` 配置

确保文件末尾有国内环境的定义（根据之前的讨论）：

```toml
[params]
  google_adsense = "ca-pub-xxxxxxxx" # 默认有广告

[environments]
  [environments.china]
    [environments.china.params]
      google_adsense = "" # 国内无广告

```

2、确认 `baseof.html` 逻辑

确保广告代码被判断语句包裹：

```html
{{ if .Site.Params.google_adsense }}
  <script ...></script>
{{ end }}

```

-----

## 背景

我想开个个人网站独立站，我想同一个站点服务既可以国内访问也可以国外访问，国外挂到国外的服务器上接谷歌的广告，这种可能国内访问比较慢就要解决国内访问慢的问题，使用同一个域名，我有域名，我有一个国内的服务器和已经备案的域名，差国外的服务器，我是个静态站点，我该怎么做

## 方案

搭建一个能够自动识别用户地理位置并分流（国内走国内服务器，国外走海外服务器）的静态网站，是一个非常经典的架构方案。这种方案不仅能提升全球访问速度，还能有效通过谷歌广告（AdSense）变现。

由于你是**静态站点**且已有**备案域名**和**国内服务器**，你可以按照以下步骤实施：

---

## 核心方案：DNS 地域线路解析 (GeoDNS)

要实现“同一个域名，不同地方访问不同服务器”，最关键的技术是 **DNS 云解析的线路细分**。

### 1. 准备国外服务器/平台

既然是静态站，你不一定非要买一台海外 VPS，可以利用全球顶级的静态托管平台，它们自带 CDN 加速：

* **推荐方案：** 使用 **Cloudflare Pages**、**Vercel** 或 **Netlify**。
* **优势：** 免费额度高，全球边缘节点分布极广，对谷歌广告（Google AdSense）非常友好。

### 2. 配置 DNS 线路分流

这是实现你需求的核心步骤。你需要将域名的解析记录（A 记录或 CNAME）按地域拆分：

| 解析线路 | 记录类型 | 记录值 (示例) | 效果 |
| --- | --- | --- | --- |
| **境内 (默认)** | A | 你的国内服务器 IP | 国内用户访问，速度快，符合备案要求 |
| **境外** | CNAME | 你的海外托管平台地址 (如 `myapp.pages.dev`) | 国外用户访问，走海外高速节点 |

> **注意：** 建议使用支持细分线路的 DNS 服务商，如 **阿里云解析 DNS**、**腾讯云 DNSPod** 或 **华为云解析**。它们都支持将流量划分为“中国内地”和“境外”。

---

## 实施路线图

### 第一步：部署海外端

1. 将你的静态代码（HTML/CSS/JS）同步到 GitHub。
2. 登录 **Cloudflare Pages**，关联该 GitHub 仓库。
3. 构建完成后，你会得到一个类似 `xxx.pages.dev` 的二级域名。

### 第二步：部署国内端

1. 将同样的代码部署到你的国内服务器。
2. 配置好 Nginx，确保通过 IP 或临时域名可以正常访问。

### 第三步：设置 DNS 分流

1. 进入你的域名解析管理后台。
2. **添加国内记录：** 线路选择“**境内**”或“**默认**”，指向国内服务器 IP。
3. **添加国外记录：** 线路选择“**境外**”，指向 Cloudflare Pages 提供的二级域名。

---

## 关键细节与注意事项

### 1. 关于谷歌广告 (AdSense)

* **统一域名：** 只要域名一致，你申请 AdSense 时填写的 `example.com` 即可。
* **ads.txt：** 确保国内和国外两台服务器的根目录下都有 `ads.txt` 文件，否则会影响广告投放。
* **国内显示问题：** 谷歌广告在国内访问通常加载极慢或无法显示。由于你分流了，国外用户能正常看广告，国内用户虽然加载慢，但因为是静态站，主要内容会先出来，广告位可能会留白或加载失败，这不影响你赚海外的钱。

### 2. 证书 (SSL/TLS) 问题

* **难点：** 两台服务器需要为同一个域名提供 HTTPS 证书。
* **解决：** 建议使用 **Let's Encrypt** 的通配符证书。你可以写一个简单的脚本，在一台服务器上续签证书后，通过 `scp` 自动同步到另一台服务器，并重启 Nginx。

### 3. 内容同步

* 既然是独立站，建议通过 **Git** 来管理。
* 利用 GitHub Actions：当你提交代码时，自动推送（Push）到国内服务器，同时触发 Cloudflare Pages 的自动构建。这样能保证两边内容实时一致。

---

### 总结建议

如果你希望操作最简单：

1. **国内：** 保持现状（备案服务器 + Nginx）。
2. **国外：** 用 Cloudflare Pages。
3. **连接点：** 在腾讯云/阿里云解析里，把“境外”线路勾选给 Cloudflare。

**你想让我帮你写一段自动同步代码到国内服务器的 GitHub Actions 脚本吗？**



---


### 总结

现在你的自动化流程是这样的：

1. GitHub Actions 使用私钥发起连接。
2. 服务器验证 `github-deploy` 用户的 `authorized_keys`，验证通过。
3. GitHub Actions 以 `github-deploy` 身份登录。
4. 因为我们做过 `chown`，该用户有权向 `/www/wwwroot/mysite` 写入文件。
5. 即使私钥泄露，黑客也只能修改你的网站文件，无法安装软件、修改系统配置或访问其他敏感目录（因为该用户没有 `sudo` 权限）。

这就是标准的“最小权限”部署配置。
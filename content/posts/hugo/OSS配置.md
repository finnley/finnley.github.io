+++
title = 'OSS配置'
date = 2025-12-14T08:14:38+08:00
draft = false
categories = [ "Hugo" ]
tags = [ "hugo", "oss" ]
+++

## 1 创建与基础配置 

### 1.1 创建 Bucket

创建Bucket，Bucket保持私有。

* **权限（ACL）：** 选择 **私有（Private）**。这意味着只有拥有密钥的请求才能访问，这是安全的基础。
* **区域：** 尽量选择离你主要受众近的区域，或者离你服务器近的区域（如果是同区域内网传输会更快）。

![alt text](/images/oss/10.png)

![alt text](/images/oss/20.png)

![alt text](/images/oss/30.png)
![alt text](/images/oss/40.png)

### 1.2 配置 CDN 加速（关键步骤）

* **不要**直接在 OSS 控制台的“域名管理”中绑定域名（那里通常用于简单的公共读 Bucket）。
* 前往 **CDN 控制台**，添加域名（例如 `static.yourblog.com`）。
* **源站设置：** 选择你的 OSS Bucket 域名。

![alt text](/images/oss/50.png)
![alt text](/images/oss/60.png)
加速区域可以暂时先选择“仅中国内地”，未来可以调整。

在“源站信息”中点击“新增源站信息”，源站信息选择“OSS域名”，域名选择第二个，之前创建Bucket分配的默认域名，不是“自定义OSS源站”，其他默认，最后点击“确定”按钮。
![alt text](/images/oss/70.png)

配置完后点击“下一步”。
![alt text](/images/oss/80.png)

进入下一步后，可以选择跳过：
![alt text](/images/oss/90.png)
![alt text](/images/oss/100.png)

接着就是根据向导提示到域名服务商去添加一条域名解析。
![alt text](/images/oss/110.png)
![alt text](/images/oss/120.png)
![alt text](/images/oss/130.png)

- 参考：https://help.aliyun.com/zh/oss/user-guide/cdn-acceleration

**源站信息是什么？**

在阿里云 CDN（或其他主流 CDN 服务）中，源站信息（Origin Server Information）是你在控制台为加速域名配置的核心部分之一。它告诉 CDN 系统：当边缘节点（CDN 节点）上没有缓存用户要访问的资源时，应该去哪里（哪个服务器）拉取最新的原始内容，即指你告诉 CDN 的真正存放文件内容的服务器地址。

简单说，就是 CDN 去哪里“拿”原始文件的地方，源站信息 = CDN 的“后勤仓库地址”。

类型,填写示例,说明,适用场景
域名,www.yourdomain.com,使用域名回源（最常见）,你的网站主域名、子域名、对象存储域名
IP地址,47.98.123.45 或 47.98.123.45:8080,直接填服务器公网 IP，可带端口,源站没有域名、或使用非标准端口
OSS域名,your-bucket.oss-cn-hangzhou.aliyuncs.com,阿里云 OSS 的内网/外网域名,使用阿里云 OSS 作为源站（最常见）
COS域名,your-bucket-1250000000.cos.ap-guangzhou.myqcloud.com,腾讯云 COS 域名,使用腾讯云 COS
多个源站,主源站 + 备源站（部分厂商支持）,配置主备切换（主挂了自动切备）,高可用场景

源站信息的作用（核心功能）

告诉 CDN 去哪里回源拉取内容
当用户第一次访问某个文件（或缓存过期、未命中），CDN 会根据你填写的“源站信息”去对应的地址请求原始文件。
决定回源的协议和端口
是 HTTP 还是 HTTPS
是否需要带端口（非 80/443）

影响回源的性能和费用
如果源站是 OSS 内网域名 → 回源走内网，零流量费用、速度快
如果源站是公网 IP 或外网域名 → 回源走公网，产生回源流量费用、延迟稍高

决定是否需要鉴权
源站是私有 OSS 桶 → 需要开启“回源鉴权”或“私有 Bucket 回源授权”
源站是公网服务器 → 可能需要源站支持 Referer、IP 白名单等防盗链

### 1.3 解决“私有”访问问题（CDN 访问 OSS 的权限）：

* 在 CDN 配置中，开启 **“私有 Bucket 回源”** 或 **“OSS 私有 Bucket 访问”** 功能（阿里云称为“阿里云 OSS 私有 Bucket 回源授权”，腾讯云也有类似设置）。
* **原理：** CDN 节点会使用一个特殊的“服务角色”身份去访问你的私有 OSS。这样，普通用户访问 CDN 域名是公开的，但 CDN 去取数据时是有权限的。

- 参考：https://help.aliyun.com/zh/cdn/user-guide/grant-alibaba-cloud-cdn-access-permissions-on-private-oss-buckets

1、在域名管理页面，单击目标域名对应的管理。
![alt text](/images/oss/140.png)

2、找到回源配置，在阿里云OSS私有Bucket回源区域打开状态。

![alt text](/images/oss/150.png)

在弹出的阿里云OSS私有Bucket回源对话框中，选择回源类型，单击确定。
![alt text](/images/oss/160.png)


## 2 安全限制（防盗链与域名限制）

我想“只能限制在我的域名下才能访问”，这正是**防盗链（Referer 引用页名单）**的功能。

- 参考：https://help.aliyun.com/zh/oss/user-guide/hotlink-protection

1、点击Bucket。
![alt text](/images/oss/170.png)


2、数据安全 > 防盗链，启用防盗链
![alt text](/images/oss/180.png)



**配置位置：** 建议在 **CDN 控制台** 配置（因为流量先到 CDN），而不是在 OSS 上配置。

1. **Referer 白名单设置：**
* **类型：** 选择“白名单”。
* **名单内容：** 填入你的博客域名，例如 `*.yourblog.com` (支持通配符) 和 `yourblog.com`。
* **策略：**
* **允许空 Referer：** **建议关闭（拒绝）**。
* *解释：* 如果允许空 Referer，用户直接在浏览器地址栏输入图片 URL 也能打开。如果你希望图片只能在博客文章里加载，**必须拒绝空 Referer**。


* **限制效果：** 配置后，只有当请求头中的 Referer 是你的博客域名时，CDN 才会返回图片；其他网站引用（盗链）或直接访问链接，都会返回 403。




2. **IP 黑/白名单（可选）：**
* 如果你发现有恶意的爬虫 IP 消耗流量，可以在 CDN 侧配置 IP 黑名单进行拦截。




2、OSS控制台绑定自定义域名（如img.yourdomain.com）。

- [通过自定义域名访问OSS-绑定域名至外网访问域名](https://help.aliyun.com/zh/oss/user-guide/access-buckets-via-custom-domain-names#1e43238a95bb6)

3、CDN控制台添加加速域名（源站设为你的OSS自定义域名）。

- [添加加速域名](https://help.aliyun.com/zh/cdn/add-a-domain-name)

4、在CDN回源配置中开启“阿里云OSS私有Bucket回源”（一键授权，CDN自动带签名回源OSS）。

> **注意** 源站信息填写的内容是OSS分配的Bucket默认域名（即像 yourbucket.oss-cn-hangzhou.aliyuncs.com 或 yourbucket.oss-cn-shanghai.aliyuncs.com 这种格式原生域名）。

- [通过CDN加速访问OSS](https://help.aliyun.com/zh/oss/user-guide/cdn-acceleration?spm=a2c4g.11186623.0.0.68185ced76CQER)
- [CDN如何使用加速域名回源OSS？](https://help.aliyun.com/zh/oss/how-does-cdn-use-an-accelerated-domain-name-to-return-to-the-source-oss)

5、配置CDN防盗链（Referer白名单：只允许你的博客域名）。

6、上传工具（如PicGo）设置URL为CDN域名或自定义域名。

7、博客Markdown中引用该URL，图片正常加载。


---
---
这是一个非常专业且符合安全最佳实践的架构思路。将 OSS Bucket 设置为**私有（Private）**，配合**CDN 加速**和**自定义域名**，是托管博客静态资源（图片、JS、CSS）的标准做法。

针对你的需求，特别是“私有 Bucket 如何对外提供服务”以及“防盗链”的问题，我为你梳理了一套完整的解决方案。

### 核心架构逻辑

由于你的 Bucket 是**默认私有读写**，直接通过 OSS 链接或直接绑定的域名访问文件会提示 `403 Forbidden`。要实现“既要私有存储，又要通过域名公开访问”，必须引入 **CDN（内容分发网络）** 作为中间层。

流程如下：
`用户请求 (blog.example.com)` -> `CDN 节点` -> `(鉴权/回源)` -> `OSS 私有 Bucket`

---

### 步骤一：创建与基础配置

1. **创建 Bucket：**
* **权限（ACL）：** 选择 **私有（Private）**。这意味着只有拥有密钥的请求才能访问，这是安全的基础。
* **区域：** 尽量选择离你主要受众近的区域，或者离你服务器近的区域（如果是同区域内网传输会更快）。


2. **配置 CDN 加速（关键步骤）：**
* **不要**直接在 OSS 控制台的“域名管理”中绑定域名（那里通常用于简单的公共读 Bucket）。
* 前往 **CDN 控制台**，添加域名（例如 `static.yourblog.com`）。
* **源站设置：** 选择你的 OSS Bucket 域名。


3. **解决“私有”访问问题（CDN 访问 OSS 的权限）：**
* 在 CDN 配置中，开启 **“私有 Bucket 回源”** 或 **“OSS 私有 Bucket 访问”** 功能（阿里云称为“阿里云 OSS 私有 Bucket 回源授权”，腾讯云也有类似设置）。
* **原理：** CDN 节点会使用一个特殊的“服务角色”身份去访问你的私有 OSS。这样，普通用户访问 CDN 域名是公开的，但 CDN 去取数据时是有权限的。



---

### 步骤二：安全限制（防盗链与域名限制）

你提到的“是否只能限制在我的域名下才能访问”，这正是**防盗链（Referer 引用页名单）**的功能。

**配置位置：** 建议在 **CDN 控制台** 配置（因为流量先到 CDN），而不是在 OSS 上配置。

1. **Referer 白名单设置：**
* **类型：** 选择“白名单”。
* **名单内容：** 填入你的博客域名，例如 `*.yourblog.com` (支持通配符) 和 `yourblog.com`。
* **策略：**
* **允许空 Referer：** **建议关闭（拒绝）**。
* *解释：* 如果允许空 Referer，用户直接在浏览器地址栏输入图片 URL 也能打开。如果你希望图片只能在博客文章里加载，**必须拒绝空 Referer**。


* **限制效果：** 配置后，只有当请求头中的 Referer 是你的博客域名时，CDN 才会返回图片；其他网站引用（盗链）或直接访问链接，都会返回 403。




2. **IP 黑/白名单（可选）：**
* 如果你发现有恶意的爬虫 IP 消耗流量，可以在 CDN 侧配置 IP 黑名单进行拦截。



---

### 步骤三：加速访问处理

为了让博客加载飞快，除了基本的 CDN 分发，还需要配置缓存策略。

1. **缓存过期时间（Cache-Control）：**
* 对于博客图片（通常不会频繁修改），建议设置较长的缓存时间。
* **文件类型：** `.jpg`, `.png`, `.webp`, `.css`, `.js`
* **过期时间：** 建议设置为 **1个月** 或 **1年**。
* *效果：* 用户第二次访问时，直接从本地浏览器缓存加载，无需消耗 CDN 流量，速度极快。


2. **开启 HTTPS/HTTP2：**
* 上传你的 SSL 证书到 CDN，开启 HTTPS 安全访问。
* 勾选 **HTTP/2** 选项，这能显著提高多图并发加载的速度。



---

### 总结与对比表

| 功能需求 | 配置位置 | 关键设置项 | 作用 |
| --- | --- | --- | --- |
| **存储安全** | OSS | ACL 设为 **私有** | 防止恶意扫描和未授权下载源文件 |
| **公开访问** | CDN | **私有 Bucket 回源授权** | 让 CDN 有权抓取私有 OSS 内容并对外分发 |
| **域名绑定** | CDN | 添加自定义域名 | 替换丑陋的 OSS 长域名，提升专业度 |
| **防盗链** | CDN | **Referer 白名单** (禁止空 Referer) | 限制资源仅能在你的博客页面内加载 |
| **访问加速** | CDN | 缓存规则、HTTP/2 | 提升加载速度，节省回源流量费用 |

### 潜在风险提示

* **流量费用：** 虽然 OSS 存储便宜，但 CDN 流量是按量计费的。如果被攻击（DDoS 或 CC），可能会产生高额账单。
* **建议方案：** 在 CDN 控制台设置 **“带宽封顶”** 或 **“用量预警”**。例如，单日流量超过 10GB 自动触发报警或停止服务，防止一夜之间房子归了云厂商。

**下一步：**
您使用的是哪家云厂商（阿里云、腾讯云、AWS S3 + CloudFront）？如果您告诉我具体的平台，我可以为您提供针对该平台具体的配置菜单路径和截图指引。
+++
title = 'SSH设置'
date = 2026-01-21T10:17:37+08:00
draft = true
categories = [ "Programming" ]
tags = [ "linux", "programming" ]
+++

**命令**
```bash
# -t 指定类型为 ed25519 (现代且安全)，-C 是注释，通常用邮箱
ssh-keygen -t ed25519 -C "your_email@example.com"
```

一路按回车键，它会在你的家目录下的 .ssh 文件夹里生成两个文件：
- id_ed25519：私钥。绝对不能泄露给任何人！ 留在你的电脑上。
- id_ed25519.pub：公钥。这个是用来放到服务器上的。

**其他**

- 将https替换为ssh
  ```
  # 将 xxx 替换为你的 GitHub 用户名
  git remote set-url origin git@github.com:xxx/repo.git
  ```

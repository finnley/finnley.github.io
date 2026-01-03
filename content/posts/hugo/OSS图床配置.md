+++
title = 'OSS配置'
date = 2025-12-14T08:14:38+08:00
draft = false
categories = [ "Hugo" ]
tags = [ "hugo" ]
+++

# 步骤

1、创建Bucket，Bucket保持私有。

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



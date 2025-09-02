+++
title = 'Vercel 部署 Hugo 站点'
date = 2024-01-16T16:29:13+08:00
draft = true
categories = [ "Hugo" ]
tags = [ "hugo" ]
+++

`Github` 由于被墙的原因在国内访问速度比较慢，`Vercel` 相比与 `GitHub` 在国内的访问速度更快，所以我打算使用 `Vercel` 来部署。

[`Vercel`](https://vercel.com/) 支持直接使用 GitHub 登录

1、登录 Vercel 后，点击页面右上角的 `Add New ...` 按钮，选择 `Project`

![](https://minio.einscat.com:19000/notes/hugo/220.png)

2、选择 GitHub 创建站点，点击 `Import`

![](https://minio.einscat.com:19000/notes/hugo/230.png)

3、部署相关配置

`Framework Preset` 选择 `Hugo`，然后点击 `Deploy` 按钮

![](https://minio.einscat.com:19000/notes/hugo/240.png)


4、部署失败，查看原因

点击红色按钮，查看详情：

![](https://minio.einscat.com:19000/notes/hugo/250.png)

![](https://minio.einscat.com:19000/notes/hugo/260.png)

原因分析：

看到错误信息中存在一些类似阿拉伯语，以为是 `hugo` 不支持，后来尝试将 `content/post/placeholder-text` 移除，因为该目录是从样例中复制过来的，里面存在阿拉伯语，移除后重新部署发现仍然失败

后面怀疑是 `Vercel` 集成的 `Hugo` 版本过低。于是在官网中找到文档 [Framework Preset](https://vercel.com/docs/deployments/configure-a-build#framework-versioning)

![](https://minio.einscat.com:19000/notes/hugo/270.png)

而我本地的 `Hugo` 版本为 `v0.121.2`，版本相差甚远。

![](https://minio.einscat.com:19000/notes/hugo/280.png)

5、阅读文档寻找答案

继续阅读文档看到下面一段介绍：

> If you would like to override Framework Preset for a specific deployment, add framework to your vercel.json configuration.

意思是我们可以通过配置 `vercel.json` 来指定特定的 `Framework Preset`

6、Vercel 指定 `Hugo` 版本

参考：

* https://blog.361way.com/2023/11/vercel-version.html
* https://vercel.com/docs/deployments/environments

在 `Hugo` 站点根目录下添加 `vercel.json` 文件，文件内容如下：

```
{
    "build": {
        "env": {
            "HUGO_VERSION": "0.121.2"
        }
    }
}
```

* `HUGO_VERSION` 版本配置按照 [Deployment Environments](https://vercel.com/docs/deployments/environments) 记录，版本号应该与 GitHub Release 版本一致，而我的版本 `0.121.2` 正好是一致的

7、重新提交到 Github，然后重新使用 `Vercel` 部署

![](https://minio.einscat.com:19000/notes/hugo/290.png)

8、设置域名

点击 `Add Domain` 进行域名设置，于是记得要进行解析哦，到服务商添加一条 CNAME 解析记录就好了。

![](https://minio.einscat.com:19000/notes/hugo/300.png)

9、通过域名进行站点预览

![](https://minio.einscat.com:19000/notes/hugo/310.png)
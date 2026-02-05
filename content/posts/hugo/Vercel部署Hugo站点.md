+++
title = 'Vercel部署Hugo站点'
date = 2024-01-16T16:29:13+08:00
draft = true
categories = [ "建站" ]
tags = [ "hugo" ]
+++

## 1 背景

`GitHub`由于被墙故在国内访问速度很慢，[`Vercel`](https://vercel.com/)相比与`GitHub`在国内的访问速度更快，所以我打算使用“Vercel”来部署。

## 2 步骤

1、登录。Vercel 支持使用GitHub登录。登录 Vercel 后，点击页面右上角的“Add New ...”按钮，选择“Project”。

![](/img/hugo/220.png)

2、导入。选择GitHub仓库后点击“Import”。

![](/img/hugo/230.png)

3、部署。“Framework Preset”下拉选择“Hugo”后点击“Deploy”按钮。

![](/img/hugo/240.png)

4、指定 Hugo 版本。在 Hugo 站点根目录下添加 `vercel.json` 文件，内容如下：
```json
{
    "build": {
        "env": {
            "HUGO_VERSION": "0.147.7"
        }
    }
}
```

要指定 Hugo 的最新版本，则将 “HUGO_VERSION” 字段的值设置为 “latest”。

参考：

* https://blog.361way.com/2023/11/vercel-version.html
* https://vercel.com/docs/deployments/environments

`HUGO_VERSION` 版本配置按照 [Deployment Environments](https://vercel.com/docs/deployments/environments) 记录，版本号应该与 GitHub Release 版本一致，而我的版本 `0.121.2` 正好是一致的。

7、重新提交到 Github，然后重新使用 `Vercel` 部署

![](/img/hugo/290.png)

8、设置域名

点击 `Add Domain` 进行域名设置，于是记得要进行解析哦，到服务商添加一条 CNAME 解析记录就好了。

![](/img/hugo/300.png)

9、通过域名进行站点预览

![](/img/hugo/310.png)


## 3 注意

我在部署过程中遇到过一些错误，当部署失败时可以点击红色按钮查看失败原因。

### 3.1 支持语言

![](/img/hugo/250.png)

原因分析：错误信息中存在一些类似阿拉伯语，文章是我复制的示例，里面存在阿拉伯语，移除后重新部署。

### 3.2 Hugo版本

![](/img/hugo/260.png)

原因分析：怀疑是 Vercel 集成的 `Hugo` 版本过低。于是在官网中找到文档 [Framework Preset](https://vercel.com/docs/deployments/configure-a-build#framework-versioning)，而我本地的Hugo版本为 “v0.121.2”，版本相差甚远。

![](/img/hugo/270.png)

![](/img/hugo/280.png)

阅读文档寻找答案，在文档看到下面一段介绍：

> If you would like to override Framework Preset for a specific deployment, add framework to your vercel.json configuration.

意思是我们可以通过配置`vercel.json`来指定特定的`Framework Preset`。

6、
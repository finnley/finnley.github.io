+++
title = 'Hugo快速上手'
date = 2024-01-13T18:14:38+08:00
draft = false
categories = [ "建站" ]
tags = [ "hugo" ]
+++

## 1 安装Hugo

- [Installation](https://gohugo.io/installation/)

## 2 安装Git

- [Install](https://git-scm.com/install/)

## 3 创建站点

1、新建站点目录。

```shell
hugo new site hugo
hugo new site finnley.github.io
```

* `hugo new site hugo` 命令中第一个 `hugo` 是命令参数，第二个 `hugo` 是站点名称，也是站点目录名称，回车后会生成一个名为“hugo”的目录，自由配置，名称自取。
* Hugo 默认配置文件格式是 `TOML`，如果想转换为 `YAML` 格式，可以指定默认配置文件格式：
    ```shell
    hugo new site hugo --format=yaml
    ```

2、初始化Git仓库。

```shell
cd hugo
git init
```

3、站点目录与文件：

```shell
archetypes
assets
content
data
hugo.toml
i18n
layouts
static
themes
```

此时如果执行 `hugo server -D` 访问页面会提示 `Page Not Found`，是因为还没有安装任何主题。

## 4 主题配置

以`hugo-theme-stack`为例。

1、添加主题，将主题作为站点的Git子目录。

```shell
git submodule add https://github.com/CaiJimmy/hugo-theme-stack/ themes/hugo-theme-stack
git submodule add https://github.com/einscat/hugo-theme-drone/ themes/hugo-theme-drone
git submodule add https://github.com/einscat/hugo-theme-drone.git themes/hugo-theme-drone
```

参考：[Stack Card-style theme designed for bloggers](https://stack.jimmycai.com/)

2、备份默认站点配置文件。

```shell
mv hugo.toml hugo.toml.backup
```

3、将 `stack` 主题样例配置文件复制到站点目录下。

```shell
cp themes/hugo-theme-stack/exampleSite/config.yaml ./
```

* 注意提供的样例配置文件格式是 `yaml` 。

4、执行 `hugo server`，预览效果。

5、预览效果。

![](/img/hugo/10.png)


5、复制样例目录中页面布局文件

```shell
cp -r themes/hugo-theme-stack/exampleSite/content/categories ./content
cp -r themes/hugo-theme-stack/exampleSite/content/page ./content
cp -r themes/hugo-theme-stack/exampleSite/content/_index.zh-cn.md ./content
cp themes/hugo-theme-stack/exampleSite/content/_index.md ./content
```

6、重启 `hugo server`，预览效果：
![](/img/hugo/20.png)

## 5 上传

```shell
cd hugo
git init
git remote add origin git@github.com:finnley/finnley.github.io.git  # 替换为自己的 Github 仓库地址
git add .
git commit -m "first commit"
git push -u origin main
```

## 6 部署

1、站点目录新建 `.github/workflows/hugo.yaml` 文件。

```shell
cd hugo
mkdir -p .github/workflows
vim .github/workflows/hugo.yaml
```

2、参考[hosting-on-github](https://gohugo.io/hosting-and-deployment/hosting-on-github/)，复制提供的样例内容到 `.github/workflows/hugo.yaml`。

3、修改`branches`中的`main`为`hugo`，如果没有新开分支，则可以不用修改。

4、推送至 Github，并按照 [hosting-on-github](https://gohugo.io/hosting-and-deployment/hosting-on-github/) 操作即可实现 Github 部署。

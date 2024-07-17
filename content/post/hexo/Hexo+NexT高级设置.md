+++
title = 'Hexo+NexT高级设置（即将废弃）'
date = 2017-04-20T11:45:03+08:00
draft = false
categories = [ "Hexo" ]
tags = [ "hexo" ]
+++

# Site

编辑 `站点配置文件`，设置 `Site` 站点信息，比如坐着，描述，语言等。

```yml
# Site
title: Finnley's Notes
subtitle: ''
description: '虽千万里，吾往矣'
keywords:
author: Finnley
language: zh-CN #en
timezone: ''
```

# Schemes

编辑 `主题配置文件`，默认是 `Muse`，喜欢哪种模式就将前面的 `#` 去掉，并在移除之前的前面加上 `#` 即可，比如我当前主题使用的是 `Gemini`。

```yml
# Schemes
#scheme: Muse
#scheme: Mist
#scheme: Pisces
scheme: Gemini
```

# Menu Settings

编辑 `主题配置文件`，找到 `menu` 一栏将要展示的菜单栏前面的 `#` 注释打开。

```yml
...
menu:
  home: / || fa fa-home
  about: /about/ || fa fa-user
  tags: /tags/ || fa fa-tags
  categories: /categories/ || fa fa-th
  archives: /archives/ || fa fa-archive
  #schedule: /schedule/ || fa fa-calendar
  #sitemap: /sitemap.xml || fa fa-sitemap
  #commonweal: /404/ || fa fa-heartbeat
```

# Sidebar Settings

编辑 `主题配置文件`，根据需要进行配置，我选择的默认设置。

```yml
sidebar:
  ...
```

# Sidebar Avatar

编辑 `主题配置文件`，设置头像链接、头像显示效果。

- url: 头像链接地址，可以是完整的互联网URI，如 `http://example.com/avatar.png`。也可以是站内的地址，如主题目录下的 `source/uploads/` （目录若不存在，新建 uploads ），配置为`url: /uploads/avatar.png`；或者放置在 `source/images/` 目录下，配置为 `url: /images/avatar.png`。

```yml
avatar:
  # Replace the default image and set the url here.
  url: https://images.notes.einscat.com/avatar.jpg #/images/avatar.gif
  # If true, the avatar will be displayed in circle.
  rounded: true #false
  # If true, the avatar will be rotated with the cursor.
  rotated: true #false
```

# Social Links

编辑 `主题配置文件`，根据需要进行配置。

```yml
social:
  GitHub: https://github.com/username || fab fa-github
  E-Mail: mailto:username@gmail.com || fa fa-envelope
  ...
```

# follow_me

```yml
follow_me:
  #Twitter: https://twitter.com/username || fab fa-twitter
  #Telegram: https://t.me/channel_name || fab fa-telegram
  WeChat: https://images.notes.xuepincat.com/wechat_channel.jpg || fab fa-weixin #/images/wechat_channel.jpg || fab fa-weixin
  #RSS: /atom.xml || fa fa-rss
```

# since

```yml
footer:
  # Specify the year when the site was setup. If not defined, current year will be used.
  since: 2017
```

# back2top

编辑 `主题配置文件`，根据需要进行配置。

- scrollpercent: 滚动时显示百分比进度

```yml
back2top:
  enable: true
  # Back to top in sidebar.
  sidebar: false
  # Scroll percent label in b2t button.
  scrollpercent: true #false
```

# Reading progress bar

编辑 `主题配置文件`，`enable` 设置为 `true`，浏览时会显示阅读进度条。

```yml
# Reading progress bar
reading_progress:
  enable: true #false
  # Available values: left | right
  start_at: left
  # Available values: top | bottom
  position: top
  reversed: false
  color: "#37c6c0"
  height: 3px
```

# codeblock

设置 code 主题为 github，另外设置 `copy_button` 的 `enable` 值为 `true` 开启复制按钮。

```yml
codeblock:
  # Code Highlight theme
  # All available themes: https://theme-next.js.org/highlight/
  theme:
    light: github #default
    dark: github #stackoverflow-dark
  prism:
    light: prism
    dark: prism-dark
  # Add copy button on codeblock
  copy_button:
    enable: true #false
    # Available values: default | flat | mac
    style:
```

# github banner

编辑 `主题配置文件`，设置 `enable` 为 `true`，在右上角区域显示 Github 跳转链接。

```yml
github_banner:
  enable: true #false
  permalink: https://github.com/yourname
  title: Follow me on GitHub
```

# mediumzoom

编辑 `主题配置文件`，设置 `enable` 为 `true`，点击图片会弹出图片预览效果

```yml
...
mediumzoom: true #false
```

# Font

编辑 `主题配置文件`，开启 `enable` 为 `true`，并设置 `family` 为 `Noto Serif SC` (宋体思源)

```yml
font:
  enable: true #false

  # Uri of fonts host, e.g. https://fonts.googleapis.com (Default).
  host: https://fonts.loli.net

  # Font options:
  # `external: true` will load this font family from `host` above.
  # `family: Times New Roman`. Without any quotes.
  # `size: x.x`. Use `em` as unit. Default: 1 (16px)

  # Global font settings used for all elements inside <body>.
  global:
    external: true
    family: Lato
    size:

  # Font settings for site title (.site-title).
  title:
    external: true
    family: Trade Winds
    size:

  # Font settings for headlines (<h1> to <h6>).
  headings:
    external: true
    family:
    size:

  # Font settings for posts (.post-body).
  posts:
    external: true
    family:

  # Font settings for <code> and code blocks.
  codes:
    external: true
    family:
```

# Local Search

添加百度/谷歌/本地 自定义站点内容搜索 [Local Search](http://theme-next.iissnan.com/third-party-services.html#local-search "Local Search") 

1.安装插件，在站点的根目录下执行以下命令：

```shell
npm install hexo-generator-searchdb --save
```

2.编辑 `站点配置文件`，添加内容如下：

```yml
# Local Search
search:
  path: search.xml
  field: post
  format: html
  limit: 10000
```

3.编辑 `主题配置文件`，启用本地搜索功能：

```yml
# Local Search
# Dependencies: https://github.com/next-theme/hexo-generator-searchdb
local_search:
  enable: true #false
```

# 设置 RSS

RSS 广泛用于网上新闻频道，blog和 wiki。使用 RSS 订阅能更快地获取信息，网站提供 RSS 输出，有利于让用户获取网站内容的最新更新。网络用户可以在客户端借助于支持 RSS 的聚合工具软件，在不打开网站内容页面的情况下阅读支持 RSS 输出的网站内容。

安装 [hexo-generator-feed](https://github.com/hexojs/hexo-generator-feed "hexo-generator-feed") 插件. 

* Install

```bash
npm install hexo-generator-feed --save
```

* Options

You can configure this plugin in _config.yml.

```yml
# RSS
feed:
  type: atom
  path: atom.xml
  limit: 20
  hub:
  content:
  content_limit: 140
  content_limit_delim: ' '
  order_by: -date
  icon: icon.png
  autodiscovery: true
  template:
```

* Configure

主题配置文件，找到 `follow_me` 相关配置，删除 `RSS` 前面的 `#`

```yml
# Subscribe through Telegram Channel, Twitter, etc.
# Usage: `Key: permalink || icon` (Font Awesome)
follow_me:
  #Twitter: https://twitter.com/username || fab fa-twitter
  #Telegram: https://t.me/channel_name || fab fa-telegram
  WeChat: https://images.notes.xuepincat.com/wechat_channel.jpg || fab fa-weixin #/images/wechat_channel.jpg || fab fa-weixin
  RSS: /atom.xml || fa fa-rss
```

# 设置「阅读全文」按钮

在文章中使用 `<!-- more -->` 手动进行截断，Hexo 提供的方式 <label style="color:white; background-color:green">推荐</label>

或者通过安装插件的方式: [hexo-auto-excerpt](https://github.com/ashisherc/hexo-auto-excerpt)

```shell
npm install --save hexo-auto-excerpt
```

# 添加字数统计、阅读时长

[hexo-wordcount](https://github.com/next-theme/hexo-word-counter)

```shell
npm install hexo-word-counter
```

主题配置文件，设置 `symbols_count_time`，开启 `separated_meta`

```yml
# Post wordcount display settings
# Dependencies: https://github.com/next-theme/hexo-word-counter
symbols_count_time:
  separated_meta: true
  item_text_total: false
```

# 发布设置

编辑 `站点配置文件`，最下面添加如下内容，托管仓库替换为自己的即可：

```yml
# Deployment
## Docs: https://hexo.io/docs/one-command-deployment
deploy:
  # type: ''
- type: git
  repository:
    github: git@github.com:finnley/finnley.github.io.git
    gitee: git@gitee.com:finnley/finnley.github.io.git
  branch: master
```

# ERROR Deployer not fond: git

* Description: 在第一次使用 `hexo d` 提交时会出现下面失败报错

![](http://images.notes.xuepincat.com/hexo/other-setting/1.jpg)

* Reason: 因为没安装 `hexo-deployer-git` 插件

* Solution: 在站点目录下输入下面的命令进行插件安装, 然后再使用 `hexo -d` 命令就可以提交了

```
npm install hexo-deployer-git --save
```

![](http://images.notes.xuepincat.com/hexo/other-setting/2.jpg)

# 提交百度谷歌收录

## 添加百度站点

1.访问 [百度站点][1] 

![](http://images.notes.xuepincat.com/hexo/other-setting/3.png)

2.添加网站

![](/images/hexo/advanced/1.png)

3.选择领域

![](http://images.notes.xuepincat.com/hexo/other-setting/5.png)

4.下载验证文件

下一步，选择文件验证，并点击 `下载验证文件`，根据页面提示步骤将文件放到指定目录中

![](/images/hexo/advanced/2.png)

这里是放在 `source` 目录中

![](http://images.notes.xuepincat.com/hexo/other-setting/7.png)

5.文件验证失败

![](/images/hexo/advanced/5.png)

6.设置 skip_render

修改 `站点配置文件` ，找到 `skip_render`，添加站点验证文件名，表示每次 `hexo g` 忽略该文件，如果不添加，每次重新生成都会将验证文件内容覆盖

![](http://images.notes.xuepincat.com/hexo/other-setting/8.png)

```yml
# Directory
source_dir: source
public_dir: public
tag_dir: tags
archive_dir: archives
category_dir: categories
code_dir: downloads/code
i18n_dir: :lang
skip_render: baidu_verify_code-3cMg3EZJT9.html
```

7.编辑 `主题配置文件`，值为 `baidu_verify_code-3cMg3EZJT9.html` 文件中的 `code-3cMg3EZJT9`

```yml
# Baidu Webmaster tools verification.
# See: https://ziyuan.baidu.com/site
baidu_site_verification: code-3cMg3EZJT9
```

8.重新验证，在第 6、7 步完成后重新上传，然后再次点击验证

![](/images/hexo/advanced/3.png)

9.完成验证

![](http://images.notes.xuepincat.com/hexo/other-setting/9.png)

稍等一会即可验证通过

![](http://images.notes.xuepincat.com/hexo/other-setting/10.jpg)

> 有3种验证方式：
> * HTML文件验证：将验证文件放置于您所配置域名的根目录下，即放在博客的本地根目录的source文件夹下（要设置skip_render）。
> * HTML标签验证：baidu_site_verification后添加HTML标签content后的内容（推荐）
> * CNAME验证：按要求添加一条CNAME解析

## 添加谷歌站点

1.登陆 [google search console](https://search.google.com/search-console/welcome?hl=zh_CN)（选右边），添加你的网站地址

![](/images/hexo/advanced/7.png)
![](/images/hexo/advanced/8.png)

2.设置-所有权验证

![](/images/hexo/advanced/9.png)

3.选择 HTML 文件验证

![](/images/hexo/advanced/10.png)

4.下载验证文件，根据页面提示步骤将文件放到指定目录中，和百度验证文件一样放入到 `source` 目录下

![](/images/hexo/advanced/11.png)

5.上传并验证，可能会出现报错

![](/images/hexo/advanced/12.png)

6.编辑 `站点配置文件`，设置 `skip_render` 添加 google 验证文件

```yml
# Directory
source_dir: source
public_dir: public
tag_dir: tags
archive_dir: archives
category_dir: categories
code_dir: downloads/code
i18n_dir: :lang
skip_render: 
  - baidu_verify_code-3cMg3EZJT9.html
  - googlef44fbd422b2411c4.html
```

7.编辑 `主题配置文件`，添加 `google_site_verification`

```yml
# Google Webmaster tools verification.
# See: https://developers.google.com/search
google_site_verification: voF4X0iKhECmUtOLCy7Jtzv7pFmAczLoeIfC-l0Nwyg
```

8.再次验证

![](/images/hexo/advanced/13.png)

## 生成sitemap站点地图

> 站点地图是一种文件，您可以通过该文件列出您网站上的网页，从而将您网站内容的组织架构告知 Google 等搜索引擎。搜索引擎网页抓取工具会读取此文件，以便更加智能地抓取您的网站。

我们需要使用插件自动生成网站的 sitemap，然后将生成的 sitemap 提交到百度和其他搜索引擎。
先安装谷歌和百度的插件

1.安装插件

* Google 版本和 Baidu 版本

```
npm install hexo-generator-sitemap --save
npm install hexo-generator-baidu-sitemap --save
```

2.修改 `站点配置文件`

在站点配置文件下添加下面代码

```
# 自动生成sitemap
sitemap:
  path: sitemap.xml
baidusitemap:
  path: baidusitemap.xml
```

3.修改 `_config.yml` 中的 `URL` 地址，把URL地址修改成你的网站地址。

![](http://images.notes.xuepincat.com/hexo/other-setting/12.png)

配置完之后执行 `hexo g` 就会在你的博客的目录 `public` 中生成相对应的sitemap文件

![](http://images.notes.xuepincat.com/hexo/other-setting/13.png)

```yml
# URL
## Set your site url here. For example, if you use GitHub Page, set url as 'https://username.github.io/project'
url: https://notes.einscat.com #http://example.com
permalink: :year/:month/:day/:title/
permalink_defaults:
pretty_urls:
...
```

修改next 主题配置文件，打开菜单字段中的站点地图：

```yml
...
menu:
  home: / || fa fa-home
  about: /about/ || fa fa-user
  tags: /tags/ || fa fa-tags
  categories: /categories/ || fa fa-th
  archives: /archives/ || fa fa-archive
  #schedule: /schedule/ || fa fa-calendar
  sitemap: /sitemap.xml || fa fa-sitemap
  #commonweal: /404/ || fa fa-heartbeat
```

4.添加 robots.txt

robots.txt 是搜索引擎蜘蛛协议，告诉引擎哪些要收录，哪些禁止收录。
source 文件夹下新建 robots.txt，内容如下:

```
User-agent: *
Allow: /
Allow: /home/
Allow: /about/
Allow: /archives/
Allow: /categories/
Allow: /tags/
Disallow: /js/
Disallow: /css/
Disallow: /fonts/
Disallow: /vendors/
Disallow: /fancybox/
Disallow: /categories/

Sitemap: https://notex.xuepincat.com/sitemap.xml
Sitemap: https://notex.xuepincat.com/baidusitemap.xml
```

4. 提交sitemap

百度：在百度站长平台--资源提交--sitemap，添加https://notes.einscat.com/baidusitemap.xml

![](/images/hexo/advanced/14.png)

对于百度，除了 sitemap 还有主动推动和自动推送这两种方式，主动推送的原理是每次 deploy 的时候都把所有链接推送给百度，自动推送则是每次网站被访问时都把该链接推送给百度。

5. 设置百度主动提交

```
npm install hexo-baidu-url-submit --save
```

上面百度站点验证通过后点击提交资源，找到 token 的值填充到下面的 `your_token` 处

![](/images/hexo/advanced/15.png)

修改站点配置文件

```yml
# 百度主动提交
baidu_url_submit:
  count: 10 ## 比如10，代表提交最新的十个链接
  host: https://notes.einscat.com ## 在百度站长平台中注册的域名
  token: your_token ## 请注意这是您的秘钥， 请不要发布在公众仓库里!
  path: baidu_urls.txt ## 文本文档的地址， 新链接会保存在此文本文档里
```

> 其中 count 表示一次推送提交最新的N个链接；host 和 token 可以在百度站点页面->数据引入->链接提交可以找到；path 为生成的文件名，里面存有推送的，我们网站的链接地址。

确保 your_site 项跟百度注册的站点一致。

同样修改站点配置文件的 deploy 项，我们原来已经有 git 的 deploy，现在增加对 baidu 的推送，最终是这样子的：

```yml
deploy:
- type: git
  repo: https://github.com/growvv/growvv.github.io.git   
  branch: master                                         

- type: baidu_url_submitter
```

重新生成发布 hexo d

另外还有自动推送，需要在主题配置文件下设置，将 `baidu_push` 设置为 `true`

```
# Enable baidu push so that the blog will push the url to baidu automatically which is very helpful for SEO.
baidu_push: true #false
```

# hexo 迁移新电脑

背景：家里有一个电脑A，在电脑A上搭建了一套hexo+next博客系统，平时也都是在这台电脑写的，但是上班工作的时候使用的是电脑B，有时候有了什么想法或者笔记的时候想记录下来，但是电脑B上没有搭建hexo环境，所以想将电脑A上面的博客迁移到电脑B上

前提：在电脑A上已经将hexo博客通过git托管在了github或者gitee上，下面所有操作都是在电脑B上完成

1.安装 nvm nodejs工具

```bash
nvm ls-remote
nvm install v16.17.0
nvm use v16.17.0
nvm current
```

2.安装hexo

```bash
npm install -g hexo-cli
```

3.clone

```bash
git clone git@github.com:finnley/finnley.github.io.git
```

4.安装依赖

```bash
cd finnley.github.io
npm install -g hexo
git checkout hexo
npm install
```

5.测试验证是否迁移成功

```bash
hexo clean
hexo version
hexo g
hexo s
```

# Baidu Analytics（百度统计）

1. 登录 [百度统计][2]，定位到站点的代码获取页面

2. 复制 `hm.js?` 后面那串统计脚本 `id`，如下图所示：

![](http://images.notes.xuepincat.com/hexo/other-setting/11.png)

3. 编辑 `主题配置文件`， 修改字段 `baidu_analytics`，值设置成你的百度统计脚本 id。

```yml
# Baidu Analytics
baidu_analytics: ff480fc7ecda6aa6118ab844a135ec83 # <app_id>
```

# Google Analytics

使用Google帐号访问 [Google Analytics](https://analytics.google.com/) ，在帐号管理中点击“创建帐号” 并进行相关设置。

```
# Google Analytics
google_analytics:
  tracking_id: UA-163505851-1 # <app_id>
  # By default, NexT will load an external gtag.js script on your site.
  # If you only need the pageview feature, set the following option to true to get a better performance.
  only_pageview: false
```

账号设置完后配置 `_config.yml` 中的 `url`

```
...
# URL
## Set your site url here. For example, if you use GitHub Page, set url as 'https://username.github.io/project'
url: https://notes.einscat.com #http://example.com
permalink: :year/:month/:day/:title/
permalink_defaults:
pretty_urls:
  trailing_index: true # Set to false to remove trailing 'index.html' from permalinks
  trailing_html: true # Set to false to remove trailing '.html' from permalinks
```

参考：https://www.webanalytics.com.cn/blog/how-to-set-up-ga-account-and-property/

# ~~添加标题翻译插件~~

hexo生成的默认文章链接格式是这样的：`https://blog.mariojd.cn/2013/07/14/<Markdown file name>/`

```
http://localhost:4000/2018/01/20/8-hexo其他配置——补缺篇/
```

这个配置在hexo站点配置文件的_config.yml里面：`permalink: :year/:month/:day/:title/`，这种默认的配置缺点很明显，当文件名是中文的时候url链接里就有中文出现，看起来low的同时也非常不利于seo优化，下面介绍两种解决方案

![](http://images.notes.xuepincat.com/hexo/other-setting/19.png)

* 解决方案

在 [hexo plugins][4]  搜索“translate”

![](http://images.notes.xuepincat.com/hexo/other-setting/20.png)

安装

```
npm install hexo-translate-title --save
```

1.配置hexo根项目下的_config.yml

```
translate_title:
  translate_way: google  # google,youdao,baidu_with_appid,baidu_no_appid
  youdao_api_key: '' # Your youdao_api_key
  youdao_keyfrom: xxxx-blog # Your youdao_keyfrom
  is_need_proxy: false     # true | false
  proxy_url: http://localhost:50018 # Your proxy_url
  baidu_appid: '' # Your baidu_appid
  baidu_appkey: '' # Your baidu_appkey
```

2.修改hexo根目录下的_config.yml

```
permalink: :year/:month:day/:translate_title.html
```

![](http://images.notes.xuepincat.com/hexo/other-setting/21.png)

# 域名绑定与解析

1. 两个解析

记录类型选 A 或 CNAME，A记录的记录值就是ip地址，github(官方文档)提供了两个IP地址，192.30.252.153 和 192.30.252.154，这两个IP地址为github的服务器地址，两个都要填上;

解析记录设置两个 notes（我使用的是二级域名，服务不是二级域名设置替换为 www） 和 @，线路就默认就行了，CNAME 记录值填你的github博客网址。如我的是 finnley.github.io。

记录类型：A
主机记录：@
解析线路：默认
记录值：192.30.252.153
TTL：10分钟

记录类型：A
主机记录：@
解析线路：默认
记录值：192.30.252.154
TTL：10分钟

记录类型：CNAME
主机记录：notes
解析线路：默认
记录值：finnley.github.io
TTL：10分钟

这些全部设置完成后，此时你并不能用申请的域名访问你的博客。接着你需要做的是在根目录的 source 文件夹里创建 CNAME 文件，不带任何后缀，里面添加你的域名信息，如：notes.xuepincat.com。
实践证明如果此时你填写的是 www.notes.xuepincat.com 那么以后你只能用 www.notes.xuepincat.com 访问，而如果你填写的是 notes.xuepincat.com。那么用 www.notes.xuepincat.com 访问也是可以的。重新清理 hexo，并发布即可用新的域名访问。

# 添加标签页面

切换到Hexo的站点目录:

!["next"](/images/hexo/next/38.png)

命令:

``` bash
$ hexo new page tags
```

回车之后会在根目录下-> `source` -> `tags` 的目录下创建一个名为 `index.md` 的文件

!["next"](/images/hexo/next/39.png)

# 添加分类页面

``` bash
$  hexo new page categories
```

!["next"](/images/hexo/next/40.png)

回车之后会在根目录下 -> `source` -> `categories` 的目录下创建一个名为 `index.md` 的文件

# 写一篇文章

输入下面命令

``` bash
$  hexo new "GitHub+Hexo搭建博客补缺篇六——NexT的简单应用"
```

!["next"](/images/hexo/next/41.png)

会在根目录 -> `source` -> `_posts` 的目录下创建一个名为 `GitHub+Hexo搭建博客补缺篇六——NexT的简单应用.md` 的文件

!["next"](/images/hexo/next/42.png)

编辑前:

!["next"](/images/hexo/next/43.png)

编辑后：

!["next"](/images/hexo/next/44.png)



分类：

!["next"](/images/hexo/next/46.png)

归档：

!["next"](/images/hexo/next/47.png)

到这里next的基本配置就全部结束了。搭建过程中也踩了一些坑，但是也从中学习到了不少的东西，可也存在这一些不完美的东西，比如博客是可以配置回复评论的，比如国内的可以使用第三方的评论系统 `多说` ，但是就在我使用的第一天，就看到多说向外宣布即将在今年（2017）的6月1号下线。

!["next"](/images/hexo/next/48.png)

虽然我与它只有“一面之缘”，但是心里也难免有些遗憾，最后祝曾经的评论系统老大“多说”一路走好。

# 文章置顶

网站看到说是要执行下面两句，但是我没有卸载原有的，我只安装了第二个 `npm install hexo-generator-index-pin-top --save`

```
npm uninstall hexo-generator-index --save
npm install hexo-generator-index-pin-top --save
```

在新增文章的开头中加入 `top: true`，比如

```
---
title: Hexo+NexT高级设置
top: true
categories: Hexo+NexT
tags:
  - Hexo
  - NexT
date: 2017-04-20 11:45:03
translate_title: hexo_next_advanced_settings
description:
---
```

# Dark Mode

原生暗黑模式缺点不支持切换，默认关闭，不用设置

```
# Dark Mode
darkmode: false
```

启用暗黑模式：[`hexo-next-darkmode`](https://github.com/rqh656418510/hexo-next-darkmode)

* Install

```
npm install hexo-next-darkmode --save
```

* Configure

You can add follow options in theme _config.yml

```
# Dark Mode
darkmode: false


# Darkmode JS
# For more information: https://github.com/rqh656418510/hexo-next-darkmode, https://github.com/sandoche/Darkmode.js
darkmode_js:
  enable: true
  bottom: '64px' # default: '32px'
  right: 'unset' # default: '32px'
  left: '32px' # default: 'unset'
  time: '0.5s' # default: '0.3s'
  mixColor: 'transparent' # default: '#fff'
  backgroundColor: 'transparent' # default: '#fff'
  buttonColorDark: '#100f2c' # default: '#100f2c'
  buttonColorLight: '#fff' # default: '#fff'
  isActivated: false # default false
  saveInCookies: true # default: true
  label: '🌓' # default: ''
  autoMatchOsTheme: true # default: true
  libUrl: # Set custom library cdn url for Darkmode.js
```

isActivated: true: default to activate darkmode, always use with saveInCookies: false and autoMatchOsTheme: false
isActivated: true：默认激活暗黑 / 夜间模式，请始终与 saveInCookies: false、autoMatchOsTheme: false 一起使用



# References

`暗黑模式`：https://www.techgrow.cn/posts/abf4aee1.html

[1]: https://ziyuan.baidu.com/site/index#/
[2]: https://tongji.baidu.com/web/10000040270/homepage/index/
[3]: https://www.google.com/webmasters/tools/
[4]: https://hexo.io/plugins/index.html

# changelog

- 2024-07-17 标题标记将废弃
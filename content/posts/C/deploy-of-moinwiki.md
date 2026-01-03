---
title: MoinWiki 部署总结
date: 2022-06-23
draft: true
categories: ["C"]
comment: false
---

MoinMoin 是一个基于Python环境的wiki引擎程序。本篇主要简单记录部署流程和遇到的坑。

# 概念介绍

部署之前先来了解几个概念：

- **WSGI**: 全称是 Python Web Server Gateway Interface（Python Web服务器网关接口），它为python web应用 和 web服务器之间的通信提供了标准接口。本质是一个通信协议。WSGI 对 Python 的意义就如同 Servlets 对 Java。
  
- **uWSGI**: uWSGI是一个是实现了WSGI协议的web服务器。

- **nginx**: Nginx是一个开源的高性能的HTTP和反向代理web服务器，支持负载均衡、高并发和高效处理静态文件。
  
一个典型的python应用的部署方式是：http 请求发送到 nginx服务器，如果请求的是静态资源，nginx可以直接读取返回资源；如果是动态资源，就转交给 uWSGI服务器，uwsgi 和 python应用通信后返回资源给uwsgi，再返回给nginx，最后由nginx返回给浏览器。

因此部署 moinwiki，我们需要做的就是安装和配置好 nginx 和 uwsgi。如何安装就不介绍了，主要贴一下两者的配置。

# moinwiki 目录结构

moin源码下载后来后，我们需要重新组织一下需要用到的文件的目录结构，然后改变相关文件中变量路径，保证和实际的路径一致。

需要关注的几个目录文件：
- wiki/data: 用户数据
- wiki/underlay: 页面支撑
- wiki/config/wikiconfig.py: wiki配置文件
- wiki/server/moin.wsgi: 与wsgi通信的文件
- MoinMoin: moinwiki源码文件
- MoinMoin/web/static/htdocs: 主题静态文件

建议将上面的5个文件或目录拉出来放到单独的文件夹里，例如 mywiki。这样我们得到的新的目录结构为:

- mywiki/MoinMoin/..
- mywiki/data/..
- mywiki/underlay/..
- mywiki/wikiconfig.py
- mywiki/moin.wsgi
- mywiki/htdocs

# moinwiki 配置

接下来修改 wikiconfig.py 的配置：必须要设置的变量有 `data_dir`, `data_underlay_dir`, 通过这两个变量找到 data 和 underlay 目录的路径。其余变量根据需求设置，每个变量配置中都有详细的解释。

重命名 moin.wsgi 为 `moin_wsgi.py`，目的是方便python执行该文件。修改 `moin_wsgi.py` 中，添加：`sys.path.insert(0, '/path/to/wikiconfigdir')` 和 `sys.path.insert(0, '/path/to/MoinSrc')`, wikiconfigdir指的是配置文件(wikiconfig.py)的目录，MoinSrc值得是moinwiki源码的目录，在我们的新的结构中就是同一个 mywiki 实例目录。如果你的结构和本文的示例不一致，修改为实际使用的目录就可以了。

通过 `moin_wsgi.py` 的设置就可以让wsgi找到我们wiki的配置文件和源码目录，然后再从配置文件中找到对应的 data 和 underlay 目录。

事实上，你可以以任意的结构组织 moinwiki 的目录，只需要在 `wikiconfig.py` 和 `moin_wsgi.py` 中配置好对应的路径，确保外围的 `uwsgi.ini` 文件可以找到 `moin_wsgi.py`，`moin_wsgi.py` 可以找到 `wikiconfig.py`。

# uWSGI 配置
在 mywiki 目录中创建 `uwsgi.ini` 文件 和 `uwsgi`目录。`uwsgi.ini`文件是uwsgi的配置文件，`uwsgi`文件夹用来存放uwsgi相关的等文件，比如日志等。


    [uwsgi]
    chmod-socket = 660
    chdir = /path/to/mywiki # moinwiki 实例目录，修改实际使用的路径
    wsgi-file = moin_wsgi.py # moinwiki和wsgi 通信的文件
    master
    workers = 3
    socket = /run/uwsgi/moin.sock # 和nginx中的配置一致
    plugins = python
    max-requests = 200
    harakiri = 30
    daemonize = %(chdir)/uwsgi/uwsgi.log
    stats = %(chdir)/uwsgi/uwsgi.status
    pidfile = %(chdir)/uwsgi/uwsgi.pid
    die-on-term

# Nginx 配置

建议在 nginx/conf.d 目录中单独建立 moin.conf 文件，该文件会自动被包含在 nginx 的主配置文件 nginx/nginx.conf 中

    upstream moin {
      server unix:/run/uwsgi/moin.sock; # 文件目录不存在时需要手动创建
    }

    server {
      listen 8088;  # 修改为实际使用的端口
      server_name 10.19.33.136;  # 修改为实际使用的服务器IP或域名

      access_log /var/log/nginx/moin.access_log main;
      error_log /var/log/nginx/moin.error_log info; # 错误日志，对于问题调试很有帮助

      location / {
        include uwsgi_params;
        uwsgi_pass moin;
      }

      location ~ ^/moin_static[0-9]+/(.*) {
        alias /path/to/mywiki/htdocs/$1; # /path/to/修改为实际目录
      }

      location /favicon.ico {
        alias /path/to/mywiki/htdocs/favicon.ico; # /path/to/修改为实际目录
      }
    }

# 运行
1. 运行uwsgi: `sudo uwsgi -d --ini uwsgi.ini`
2. 运行nginx: `sudo systemctl start nginx`
3. 停止uwsgi: `sudo pkill -f uwsgi -9`
4. 重启nginx: `sudo systemctl restart nginx`
5. 检查uwsgi进程: `ps aux | grep uwsgi`
6. 检查ngnix状态: `sudo systemctl status nginx`

每次修改了配置，需要重启 uwsgi 和 nginx。

# 总结
1. 官方文档：https://moinmo.in/ 。
2. 遇到问题，多看日志。本例中，uwsgi的日志是 `/path/to/mywiki/uwsgi/uwsgi.log`；nginx的日志是 `/var/log/nginx/moin.error_log`。
3. 日志中出现权限问题，除了是文件本身权限问题，检查一下相关目录是否已经创建。比如本例中的 `/run/uwsgi/`, `/path/to/mywiki/uwsgi`等，请务必检查已经创建。

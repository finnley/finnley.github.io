+++
title = '宿主机Mac无法Ping通容器IP'
date = 2024-11-03T12:26:48+08:00
draft = false
categories = [ "Mac" ]
tag = [ "mac", "docker" ]
+++

# 背景

- 宿主机：Mac
- 容器：在Mac上安装的 CentOS8 容器，容器IP为 172.20.30.1
- 现象：进入容器中，可以Ping通宿主机Mac的IP，但在宿主机Mac上却无法Ping通容器的IP，提示信息如下：
    ```bash
    ✗ ping 172.20.30.1
    PING 172.20.30.1 (172.20.30.1): 56 data bytes
    Request timeout for icmp_seq 0
    Request timeout for icmp_seq 1
    ```

# 解决方案

## 参考

- [关于Mac宿主机无法ping通Docker容器的问题](https://www.cnblogs.com/luo-c/p/15830769.html)
- [Docker for Mac 的网络问题及解决办法（新增方法四）](https://www.haoyizebo.com/posts/fd0b9bd8/)
- [desktop-docker-connector](https://github.com/wenjunxiao/mac-docker-connector/blob/master/README-ZH.md)

## 我的

**1、首先 Mac 端通过 brew 安装 docker-connector**

保留重要信息如下：
```bash
✗ brew install wenjunxiao/brew/docker-connector
...
==> Tapping wenjunxiao/brew
Cloning into '/opt/homebrew/Library/Taps/wenjunxiao/homebrew-brew'...
remote: Enumerating objects: 73, done.
remote: Counting objects: 100% (73/73), done.
remote: Compressing objects: 100% (49/49), done.
remote: Total 73 (delta 38), reused 44 (delta 17), pack-reused 0 (from 0)
Receiving objects: 100% (73/73), 11.35 KiB | 2.84 MiB/s, done.
Resolving deltas: 100% (38/38), done.
Tapped 2 formulae (16 files, 24.6KB).
==> Fetching wenjunxiao/brew/docker-connector
==> Downloading https://github.com/wenjunxiao/mac-docker-connector/releases/download/v3.1/docker-connector-darwin.tar.gz
==> Downloading from https://objects.githubusercontent.com/github-production-release-asset-2e65be/266031479/3f51cb4b-e37f-4f12-a4
########################################################################################################################## 100.0%
==> Installing docker-connector from wenjunxiao/brew
Warning: A newer Command Line Tools release is available.
Update them from Software Update in System Settings.

If that doesn't show you any updates, run:
  sudo rm -rf /Library/Developer/CommandLineTools
  sudo xcode-select --install

Alternatively, manually download them from:
  https://developer.apple.com/download/all/.
You should download the Command Line Tools for Xcode 15.2.

==> Caveats
For the first time, you can add all the bridge networks of docker to the routing table by the following command:
  docker network ls --filter driver=bridge --format "{{.ID}}" | xargs docker network inspect --format "route {{range .IPAM.Config}}{{.Subnet}}{{end}}" >> /opt/homebrew/etc/docker-connector.conf
Or add the route of network you want to access to following config file at any time:
  /opt/homebrew/etc/docker-connector.conf
Route format is `route subnet`, such as:
  route 172.17.0.0/16
The route modification will take effect immediately without restarting the service.
You can also expose you docker container to other by follow settings in /opt/homebrew/etc/docker-connector.conf:
  expose 0.0.0.0:2512
  route 172.17.0.0/16 expose
Let the two subnets access each other through iptables:
  iptables 172.17.0.0+172.18.0.0

To start wenjunxiao/brew/docker-connector now and restart at login:
  brew services start wenjunxiao/brew/docker-connector
Or, if you don't want/need a background service you can just run:
  sudo /opt/homebrew/opt/docker-connector/bin/docker-connector -config /opt/homebrew/etc/docker-connector.conf
==> Summary
🍺  /opt/homebrew/Cellar/docker-connector/3.1: 6 files, 5.4MB, built in 1 second
==> Running `brew cleanup docker-connector`...
Disable this behaviour by setting HOMEBREW_NO_INSTALL_CLEANUP.
Hide these hints with HOMEBREW_NO_ENV_HINTS (see `man brew`).
==> `brew cleanup` has not been run in the last 30 days, running now...
Disable this behaviour by setting HOMEBREW_NO_INSTALL_CLEANUP.
...
```

或者
```bash
$ brew tap wenjunxiao/brew
$ brew install docker-connector
```

**2、通过以下命令把所有Docker所有bridge子网放入配置文件，后续的增减可以参考后面的详细配置**
```bash
✗ docker network ls --filter driver=bridge --format "{{.ID}}" | xargs docker network inspect --format "route {{range .IPAM.Config}}{{.Subnet}}{{end}}" >> "$(brew --prefix)/etc/docker-connector.conf"
```

执行完后查看 `/opt/homebrew/etc/docker-connector.conf` 文件，该文件在安装 `wenjunxiao/brew/docker-connector` 时的返回信息中提到，见最后三行，最后两行是我容器环境的IP网段。
```bash
route 172.17.0.0/16
route 172.20.30.0/24
route 172.20.40.0/24
```

**3、启动Mac端的服务**
```bash
✗ brew services start wenjunxiao/brew/docker-connector
==> Tapping homebrew/services
Cloning into '/opt/homebrew/Library/Taps/homebrew/homebrew-services'...
remote: Enumerating objects: 3487, done.
remote: Counting objects: 100% (633/633), done.
remote: Compressing objects: 100% (248/248), done.
remote: Total 3487 (delta 451), reused 477 (delta 381), pack-reused 2854 (from 1)
Receiving objects: 100% (3487/3487), 1019.83 KiB | 604.00 KiB/s, done.
Resolving deltas: 100% (1692/1692), done.
Tapped 2 commands (52 files, 1.2MB).
==> Successfully started `docker-connector` (label: homebrew.mxcl.docker-connector)
```

**4、安装Docker端的容器mac-docker-connector**
```bash
✗ docker pull wenjunxiao/mac-docker-connector
✗ docker run -it -d --restart always --net host --cap-add NET_ADMIN --name mac-connector wenjunxiao/mac-docker-connector
Unable to find image 'wenjunxiao/mac-docker-connector:latest' locally
latest: Pulling from wenjunxiao/mac-docker-connector
26d14edc4f17: Pull complete
8190e2a13d0f: Pull complete
Digest: sha256:3408a58f96d7dccf28df68f422ce215a4a21d5e8302aee2e8c23acc2feab4948
Status: Downloaded newer image for wenjunxiao/mac-docker-connector:latest
ef1d98ebf2a196e61333899fdeb628d12925c64eaa8bb821531a76312d8d8cd2
```

**5、重新在宿主机上Ping容器IP**
```bash
✗ ping 172.20.30.1
PING 172.20.30.1 (172.20.30.1): 56 data bytes
64 bytes from 172.20.30.1: icmp_seq=0 ttl=63 time=0.599 ms
64 bytes from 172.20.30.1: icmp_seq=1 ttl=63 time=1.210 ms
64 bytes from 172.20.30.1: icmp_seq=2 ttl=63 time=0.755 ms
```

**说明**
1. 在这过程中也并非一帆风顺，安装 `docker-connector` 并非直接执行【2】的命令，而是手动配置的 `/opt/homebrew/etc/docker-connector.conf` 文件，内容为如下。然后继续后面的操作，结果还是不能成功Ping通。
    ```bash
    # addr 192.168.251.1/24
    # mtu 1400
    # host 127.0.0.1
    # port 2511
    # route 172.17.0.0/16
    # route 172.18.0.0/16
    route 172.18.0.0/24
    # iptables 172.17.0.0+172.18.0.0
    # hosts /etc/hosts .local
    # proxy 127.0.0.1:80:80
    ```
2. 期间还有出现执行 `brew services start wenjunxiao/brew/docker-connector` 失败的情况，后面反复了几次终于不再出现错误信息。
3. 另外还有在配置完后还有重启了宿主机的操作。
4. 最后是看到参考 Github 后改用命令添加IP的方式才得以解决。

# 小结

- 需了解 `Docker Desktop for Mac and Windows` 为何没有提供从宿主的 macOS 或 Windows 通过容器IP访问容器的方式。
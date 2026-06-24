---
title: Docker 使用代理拉取镜像
date:  2026-06-22 22:28:17

math: false
mermaid: false
category_bar: false

categories:
  - Linux

tags:
  - Proxy
  - Docker

excerpt: "使用代理拉取 Docker 的镜像"
permalink: /posts/20260622-222817.html
---
## 1. 环境

- Ubuntu 22.04 LTS

## 2. 系统级服务代理

通常，可以使用 `sudo -E` 传递代理环境变量，但是 Docker 采用的是 C/S (客户端/服务端) 架构。`docker pull` 只是客户端。真正负责联网下载镜像的是运行在后台的 Docker Daemon 服务端（dockerd 进程）。使用 `sudo -E` 仅仅把代理环境变量传递给了 Docker 客户端。后台的 dockerd 进程根本没有获取到这些代理配置，因此它依然在尝试直连公网，最终导致 DNS 解析超时。

为此需要直接为 Docker Daemon 服务配置代理。以下是标准的系统级配置方法。

### 2.1 代理配置文件

```Bash
# 创建 Docker 服务配置目录
sudo mkdir -p /etc/systemd/system/docker.service.d

# 创建代理配置文件
sudo vim /etc/systemd/system/docker.service.d/http-proxy.conf
```

在文件中添加以下内容：

```Ini
[Service]
Environment="HTTP_PROXY=http://127.0.0.1:10808"
Environment="HTTPS_PROXY=http://127.0.0.1:10808"
Environment="NO_PROXY=localhost,127.0.0.1,.domain.com"
```

{% note warning %}
请务必将上面的协议和端口替换为实际的代理地址。

注：`NO_PROXY` 很重要，它能确保拉取本地或内网私有仓库镜像时不经过代理。
{% endnote %}

### 2.2 重载配置

让 systemd 重新读取配置，并重启 docker 服务使其生效：

```Bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```

### 2.3 测试

在重新执行拉取命令之前，先验证一下 Docker Daemon 是否已经成功加载了代理：

```Bash
sudo docker info | grep Proxy
```

成功加载代理后，应输出：

```Text
HTTP Proxy: http://127.0.0.1:10808
HTTPS Proxy: http://127.0.0.1:10808
No Proxy: localhost,127.0.0.1,.domain.com
......
```

{% note success %}
之后就可以使用 `sudo docker pull` 拉取镜像了！
{% endnote %}

{% note info %}
配置 Docker Daemon 代理仅对 拉取/推送镜像 (docker pull / docker push) 生效。它不会让 容器内部 (docker run) 或 构建过程 (docker build) 走代理。如果需要让容器内联网也走代理，需要修改 `~/.docker/config.json` 或在运行/构建时通过 `--env` 或 `--build-arg` 显式传递参数。
{% endnote %}

## 3. 现代配置方法

从 Docker Engine 23.0 开始，官方原生支持在 `/etc/docker/daemon.json` 中直接配置代理，无需再去修改 systemd 文件。

创建或者修改文件：

```Bash
sudo vim /etc/docker/daemon.json
```

添加以下内容：

```Json
{
  "proxies": {
    "http-proxy": "http://127.0.0.1:10808",
    "https-proxy": "http://127.0.0.1:10808",
    "no-proxy": "localhost,127.0.0.1,.domain.com"
  }
}
```

{% note warning %}
请务必将上面的协议和端口替换为实际的代理地址。
{% endnote %}

应用配置：

```Bash
sudo systemctl restart docker
```

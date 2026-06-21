---
title: Python sanity check failed 报错解决办法
date:  2026-06-07 18:20:21

math: false
mermaid: false
category_bar: false

categories:
  - Linux

tags:
  - ESP-IDF
  - Proxy

excerpt: "使用 eim-cli 安装 ESP-IDF 时报错"
permalink: /posts/20260607-182021.html
---
## 1. 环境

- Ubuntu Server 24.04.4 LTS

## 2. 报错信息

由于需要拉取 GitHub 仓库，我在远程服务器上通过 SSH 反向隧道映射了本地代理。但在使用 `eim wizard` 安装时，环境无法通过测试，报错如下：

```Text
[FAIL] SSL/HTTPS
    Hint: Check your network and proxy settings, or try to install OpenSSL development headers: sudo apt install libssl-dev (Debian/Ubuntu), then reinstall Python
Error executing CLI: Python sanity check failed. See the [FAIL] entries above for details. 
```

花了一段时间排查后，发现在配置了全局代理后，Python 的网络底层库会默认将发往 localhost 或 127.0.0.1 的自环流量也转发给代理服务器，从而导致路由失败或连接被拒。

## 3. 解决办法

在 Ubuntu 上配置代理时，加上白名单即可：

```Bash
# 绕过本地流量
export NO_PROXY="localhost,127.0.0.1,0.0.0.0,::1,.local"
export no_proxy=$NO_PROXY
```

这样本地和内网流量就会直连，不走代理。
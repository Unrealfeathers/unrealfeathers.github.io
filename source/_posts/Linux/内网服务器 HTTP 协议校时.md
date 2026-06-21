---
title: 内网服务器 HTTP 协议校时
date:  2026-06-18 20:31:44

math: false
mermaid: false
category_bar: false

categories:
  - Linux

tags:
  - Network
  - Proxy
  - SSH

excerpt: "内网服务器无法访问 NTP 服务器，使用 HTTP 协议进行低精度校时"
permalink: /posts/20260618-203144.html
---
## 1. 环境

- Ubuntu 22.04 LTS

## 2. 原理

任何标准的 Web 服务器在其 HTTP 响应头中都会包含一个 `Date` 字段。因此可以使用这个时间进行校时，精度不高，但比手动强。

若服务器完全无公网出口，可先通过 SSH 反向隧道建立临时代理，操作可参考以下文章：

{% note info %}
[通过 SSH 将本地代理反向隧道映射到云服务器 | Unrealfeathers' Blog](https://flowerdown.org/posts/20260606-141629)
{% endnote %}

## 3. 步骤

在内网服务器上，通过代理访问一个稳定的大型网站，抓取时间头并同步给系统：

```Bash
# 获取并解析时间头
sudo date -s "$(curl -x http://127.0.0.1:10808 -sI https://www.bing.com | grep -i '^date:' | sed 's/^[Dd]ate: //g')"

# 将当前系统时钟的时间写入硬件时钟(RTC)
sudo hwclock -w
```

{% note info %}
代理端口 `10808` 请根据自身配置进行对应修改。
{% endnote %}

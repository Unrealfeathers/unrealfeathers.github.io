---
title: 配置代理后 sudo apt update 失败
date:  2026-06-20 15:32:43

math: false
mermaid: false
category_bar: false

categories:
  - Linux

tags:
  - Proxy

excerpt: "Ubuntu 22.04 LTS 配置代理后，更新软件包列表失败"
permalink: /posts/20260620-153243.html
---
## 1. 环境

- Ubuntu 22.04 LTS

## 2. 报错信息

执行 `sudo apt update` 后，发现软件源均无法连接：

```Text
Ign:1 https://mirrors.ustc.edu.cn/ubuntu jammy InRelease
......
```

但是使用 `curl` 命令测试，可以正常连接：

```Bash
curl -I https://mirrors.ustc.edu.cn
```

输出：

```Text
HTTP/1.1 200 Connection established

HTTP/2 200
......
```

## 3. 解决办法

`curl` 能够正常握手，说明代理在当前用户环境下是正常工作的。而 `sudo apt update` 卡住，是因为当使用 `sudo` 提权执行命令时，`sudo` 出于安全考虑，默认会重置并清空当前用户的环境变量。因此，通过 `export` 设置的代理变量对 root 权限下的 `apt` 根本没有生效。

### 3.1 临时配置

如果只是临时使用代理的话，可以加上 `-E` 参数，其作用是保留当前用户的环境变量。

```Bash
sudo -E apt update
```

也可以仅传递特定的代理环境变量：

```Bash
sudo http_proxy="http://127.0.0.1:10808" https_proxy="http://127.0.0.1:10808" apt update
```

`apt` 的其它操作同理。

{% note info %}
代理端口 `10808` 请根据自身配置进行对应修改。
{% endnote %}

### 3.2 永久配置

如果希望 `apt` 每次运行都自动走代理，而不需要每次都加 `-E`，最好的做法是直接修改 `apt` 的配置文件。

创建或编辑一个 apt 配置文件：

```Bash
sudo vim /etc/apt/apt.conf.d/proxy.conf
```

在文件中加入以下内容：

```Text
Acquire::http::Proxy "http://127.0.0.1:10808/";
Acquire::https::Proxy "http://127.0.0.1:10808/";
```

{% note info %}
代理端口 `10808` 请根据自身配置进行对应修改。
{% endnote %}

配置完成后，直接运行 `sudo apt update` 即可。

{% note warning %}
此配置在关闭代理后，会导致 `apt` 报错无法使用。如需取消代理恢复直连，只需将对应的配置文件删除或将文件内的内容注释掉即可。
{% endnote %}

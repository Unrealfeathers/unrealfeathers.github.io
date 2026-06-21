---
title: 通过 SSH 将本地代理反向隧道映射到云服务器
date:  2026-06-06 14:16:29

math: false
mermaid: false
category_bar: false

categories:
  - Linux

tags:
  - Network
  - Proxy
  - SSH

excerpt: "出于水表安全，最好不要在大陆的云服务器上挂代理"
permalink: /posts/20260606-141629.html
---
## 1. 环境

- 本地 Windows 10 22H2
- 云服务器 Ubuntu Server 24.04.4 LTS

## 2. 建立 SSH 反向隧道

我们可以通过配置 SSH 客户端，在连接服务器时自动建立一条反向隧道，将云服务器上的流量转发回本地主机。

打开或者创建本地 Windows 的 SSH 配置文件，路径为：`C:\Users\username\.ssh\config`，`username` 为你的 Windows 用户名。

在文件中添加以下内容：

```Text
# xxx.xxx.xxx.xxx 为云服务器 IP
Host xxx.xxx.xxx.xxx
  HostName xxx.xxx.xxx.xxx
  # username 为云服务器用户名
  User username
  # 反向转发配置, 将远程 10808 端口映射到本地的 10808 端口
  RemoteForward 10808 127.0.0.1:10808
```

保存后，只要通过 SSH 连接此云服务器，这份配置就会生效，反向隧道会自动在后台默默打通。

## 3. 本地主机配置代理

我这里使用的是 v2rayN，本地混合监听地址为 10808，并开启自动配置系统代理。由于反向隧道是将云服务器的流量转发到本地的 127.0.0.1，因此 v2rayN 默认设置即可接收这些请求，无需开启“允许局域网连接”。

## 4. 云服务器配置代理

### 4.1 防火墙放行

由于反向隧道的所有流量都被封装在已经建立的 SSH 连接中，并且在云服务器上映射的地址是本地环回地址 127.0.0.1，因此不需要在云服务器的安全组或防火墙中单独放行 10808 端口。这天然保证了代理端口不会暴露在公网，非常安全。

### 4.2 配置云服务器代理

为了方便随时开启和关闭代理，可以把环境变量配置写进 `~/.bashrc` 中，作为快捷命令使用。

首先，编辑配置文件：

```Bash
# 编辑文件
vim ~/.bashrc
```

在文件末尾添加以下内容：

```Bash
# 开启代理的快捷命令
function proxy_on() {
    export http_proxy="http://127.0.0.1:10808"
    export https_proxy="http://127.0.0.1:10808"
    export all_proxy="socks5://127.0.0.1:10808"

    # 同时设置大写，兼容所有软件
    export HTTP_PROXY=$http_proxy
    export HTTPS_PROXY=$https_proxy
    export ALL_PROXY=$all_proxy

    # 绕过本地流量
    export NO_PROXY="localhost,127.0.0.1,0.0.0.0,::1,.local"
    export no_proxy=$NO_PROXY

    echo "终端代理已开启"
}

# 关闭代理的快捷命令
function proxy_off() {
    unset http_proxy https_proxy all_proxy
    unset HTTP_PROXY HTTPS_PROXY ALL_PROXY
    unset NO_PROXY no_proxy
    echo "终端代理已关闭"
}
```

保存退出后，使配置立即生效：

```bash
# 应用配置
source ~/.bashrc
```

### 4.3 测试代理连通性

在云服务器终端输入 `proxy_on` 开启代理，然后使用 `curl` 命令测试访问外部网络：

```Bash
#开启代理
proxy_on

#测试
curl -I https://www.google.com
```

输出如下视为正常：

```Text
HTTP/1.1 200 Connection established

HTTP/2 200
......
......
```

{% note success %}
之后就可以快乐的在云服务器上使用官方源和 Github 了！
{% endnote %}
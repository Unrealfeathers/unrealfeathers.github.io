---
title: 使用 Frp 进行 SSH 连接
date:  2026-05-23 18:57:29

math: false
mermaid: false
category_bar: false

categories:
  - Linux

tags:
  - Frp
  - SSH

excerpt: "使用 Frp 来让本地电脑 SSH 到远程 Windows 11 主机上的 Ubuntu 虚拟机"
permalink: /posts/20260523-185729.html
---
## 1. 环境

- 本地 Windows 10 22H2
- 远程 Windows 11 25H2
- 虚拟机 Ubuntu Desktop 24.04.4 LTS
- 云服务器 Ubuntu Server 24.04.4 LTS

## 2. 准备条件

> [Releases · fatedier/frp](https://github.com/fatedier/frp/releases)

1. 下载对应系统版本的 frp 软件；
2. 请在云服务器的控制台确保 7000 和 6000 端口已经放行，两个端口均使用 TCP 协议；
3. 确保远程 Windows 主机可以 ping 通虚拟机的 IP，并可以进行 SSH 连接；

## 3. 部署 frp

### 3.1 服务端

1. 使用 `scp` 命令将压缩包传到云服务器上，并解压到家目录下；

2. 进入 frp 文件目录，编辑 `frps.toml` 文件：

```Toml
# frp 客户端与服务端通信的端口
bindPort = 7000
```

3. 启动服务端：

```Bash
# 新建会话
tmux new -s frp

# 运行服务
./frps -c ./frps.toml

# 先按下 ctrl + b，再按下 d 退出会话
```

{% note success %}
注：tmux 的详细使用可以参考 [tmux 的使用 | Unrealfeathers' Blog](https://flowerdown.org/posts/20260529-193555)
{% endnote %}

### 3.2 客户端

1. 上传对应版本的 frp到远程 Windows 主机，并解压压缩包；

2. 使用记事本编辑 `frpc.toml` 文件：

```Toml
serverAddr = "x.x.x.x" # 云服务器的公网 IP
serverPort = 7000          # 必须与 frps.toml 中的 bindPort 一致

[[proxies]]
name = "ssh"
type = "tcp"
localIP = "192.168.1.2"    # Ubuntu 虚拟机的 IP，千万不要填 127.0.0.1
localPort = 22             # Ubuntu 虚拟机的 SSH 端口
remotePort = 6000          # 暴露在云服务器上的端口
```

3. `win + r` 打开运行，输入 `powershell` 然后回车。进入到 frp 文件目录，输入以下命令运行 frp：

```Powershell
.\frpc.exe -c .\frpc.toml
```

## 4. 远程连接

现在云服务器的 6000 端口已经被映射到了远程 Ubuntu 虚拟机的 22 端口。在本地电脑上，打开终端，输入以下命令：

```Powershell
ssh username@x.x.x.x -p 6000
```

- `username`：远程 Ubuntu 虚拟机的用户名；
- `x.x.x.x`：云服务器的公网 IP；
- `-p 6000`：在 frpc.toml 中配置的 remotePort；

## 5. 远程文件传输

1. 从本地电脑上传文件到远程 Ubuntu 虚拟机。把本地的 test.txt 上传到 Ubuntu 的 /home/username/ 目录下：

```Bash
scp -P 6000 /本地文件的路径/test.txt username@x.x.x.x:/home/username/
```

2. 从远程 Ubuntu 虚拟机下载文件到本地电脑。把 Ubuntu 上的 data.log 下载到本地当前目录：

```Bash
scp -P 6000 username@x.x.x.x:/home/username/data.log /本地保存的路径/
```

3. 传输整个文件夹。把本地的 my_folder 文件夹整个传到 Ubuntu 虚拟机：

```Bash
scp -P 6000 -r /本地/my_folder username@x.x.x.x:/home/username/
```
---
title: 使用 tcpdump 进阶抓包
date:  2026-05-30 12:58:05

math: false
mermaid: false
category_bar: false

categories:
  - Linux

tags:
  - Network
  - tcpdump
  - UDP

excerpt: "如何使用 tcpdump 抓取指定 IP 和端口的数据包，并分析 UDP 载荷及特定字节的过滤规则"
permalink: /posts/20260530-125805.html
---
## 1. 基础抓包

```Bash
sudo tcpdump -i any -nn -X 'dst host 192.168.1.2 and dst port 2233'
```

- `-i any`： 监听所有网卡；

- `-nn`： 不解析主机名和端口服务名， 强制显示 IP 地址和端口号；

- `-X`： 同时以十六进制 (Hex) 和 ASCII 码打印数据包的 payload 以及 IP 包头；

- `'dst host 192.168.1.2 and dst port 2233'`： 抓取目的 IP 和端口是 192.168.1.2：2233 的数据包；

如果流量过大，建议将 `-nn -X` 替换为 `-w capture.pcap`，将数据包保存为文件后使用 Wireshark 进行图形化分析：

```Bash
sudo tcpdump -i any -w capture.pcap 'dst host 192.168.1.2 and dst port 2233'
```

- `-w capture.pcap`： 将数据包抓取并保存为 .pcap 文件；

## 2. UDP 数据包结构

### 2.1 寻找真正的 UDP Payload

如果只看 UDP 协议本身，它的头部固定为 8 个字节，真正的载荷从第 9 个字节开始。但在 `tcpdump -X` 的输出中，我们看到的是经过封装的完整数据包，需要考虑底层协议的头部。

- 标准以太网抓包：以太网头(14) + IPv4头(20) + UDP头(8) = 42 字节，载荷从第 43 字节开始；

- 网络层抓包：IPv4头(20) + UDP头(8) = 28 字节，载荷从第 29 字节开始；

- Linux Cooked Capture：SLL头(16) + IPv4头(20) + UDP头(8) = 44 字节，载荷从第 45 字节开始；

### 2.2 抓包开头的 "4500"

当查看 tcpdump 的十六进制输出时，如果第一行以 4500 开头，这是一个极其重要的特征。它表明 tcpdump 跳过了链路层 (MAC 层)，直接从网络层 (IP 层) 开始展示数据。

- 0x45：前 4 位 4 代表 IPv4；后 4 位 5 代表头部长度是以 32-bit (4字节) 为单位的，5 * 4 = 20 字节，这是标准 IPv4 头部的固定长度；

- 0x00：代表服务类型 (ToS / DiffServ) 为默认值，无特殊优先级标记；

## 3. BPF 字节过滤语法

假设我们需要精确抓取 UDP Payload 的第 6 个字节等于 0x08 的数据包，可以利用 tcpdump 的 BPF 语法来实现。假设抓包数据以 4500 开头，即从 IP 层开始。我们可以直接以 IP 层作为基准进行偏移量过滤。

```Bash
sudo tcpdump -i any -nn -X 'dst host 192.168.1.2 and dst port 2233 and ip[33] == 0x08'
```

- `ip[33] == 0x08`： 要求第 34 个字节等于 0x08，IPv4头(20) + UDP头(8) + Payload(6) = 34 字节；

此外，UDP 协议还可以直接使用 `udp[13] == 0x08` (UDP头(8) + Payload(6) = 13) 代替 `ip[33] == 0x08`，这样可以自动跳过变长的 IP 头部，直接定位到 UDP 头部。

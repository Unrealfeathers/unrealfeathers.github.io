---
title: 使用 nftables 进行 UDP 数据包拦截
date:  2026-06-26 17:26:13

math: false
mermaid: false
category_bar: false

categories:
  - Linux

tags:
  - UDP
  - nftables

excerpt: "使用 nftables 拦截具有指定内容的 UDP 数据包"
permalink: /posts/20260626-172613.html
---
## 1. 抓包

可以使用 `tcpdump` 抓取指定的 UDP 数据包，可以参考以下文章：

> [使用 tcpdump 进阶抓包 | Unrealfeathers' Blog](https://flowerdown.org/posts/20260530-125805)

## 2. nftables

`nftables` 支持直接在传输层头部进行位偏移匹配，其逻辑与 `tcpdump` 的 `udp[13]` 完全一致。在 `tcpdump` 中，`udp[13]` 指的是从 UDP 头部开始偏移 13 个字节（即 UDP 头 8 字节 + 负载第 5 个字节）。

在 `nftables` 中，可以使用 `@th` (Transport Header) 来定位，公式为：`13 字节 × 8 比特/字节 = 104 比特` 偏移量。读取长度为 8 比特（1 字节），值为 0x01。

在操作前，请确保系统没有运行与 `nftables` 冲突的传统 `iptables` 规则或 `ufw` 防火墙策略，以免流量被提前拦截或放行。

### 2.1 创建独立表

1. 创建一个独立表：

```Bash
sudo nft add table inet custom_filter
```

2. 创建挂载在 `output` / `input` (处理本机 发送/接收 的包) 钩子上的链：

```Bash
sudo nft add chain inet custom_filter output_chain { type filter hook output priority 0 \; policy accept \; }
# or
sudo nft add chain inet custom_filter input_chain { type filter hook input priority 0 \; policy accept \; }
```

{% note warning %}
注：如果本机是一台路由器，包是从其他机器经过它转发的，请将 `hook output` 改为 `hook forward`。若将钩子改为 `hook forward`，建议同时将命令中的链名称也更改为 `forward_chain`，以保持命名与语义的一致性。
{% endnote %}

### 2.2 添加精准拦截规则

1. 拦截本机发送的包：

```Bash
sudo nft add rule inet custom_filter output_chain ip daddr 192.168.1.2 udp dport 2233 @th,104,8 == 0x01 counter drop
```

{% note info %}
这里 `ip daddr 192.168.1.2` 代表目的 IP 是 192.168.1.2，`udp dport 2233` 代表目标端口是 2233，`@th,104,8 == 0x01` 计算方式请参考本章开头。
{% endnote %}

2. 拦截本机接收的包：

```Bash
sudo nft add rule inet custom_filter input_chain ip saddr 192.168.1.3 udp dport 3322 @th,104,8 == 0x01 counter drop
```

{% note info %}
这里 `ip saddr 192.168.1.3` 代表源 IP 是 192.168.1.3，`udp dport 3322` 代表目标端口（本机）是 3322。
{% endnote %}

### 2.3 测试

运行 `sudo nft list table inet custom_filter`，会看到 `counter` 的数值在增加，说明数据包正在被成功拦截并丢弃。

### 2.4 删除规则

如果需要取消拦截，直接删除独立表即可：

```Bash
sudo nft delete table inet custom_filter
```

此外也可以通过句柄只删除特定规则，而不影响表内的其他规则：

```Bash
# 查找句柄
sudo nft -a list table inet custom_filter

# 按句柄删除, X->句柄
sudo nft delete rule inet custom_filter input_chain handle X
```

### 2.5 规则持久化

当前给出的 `nft` 命令都是临时生效的，重启后规则会丢失。如果需要将规则持久化，可以参照以下命令：

```Bash
# 保存规则
sudo nft list ruleset | sudo tee /etc/nftables.conf > /dev/null

# 检查配置
sudo nft -c -f /etc/nftables.conf

# 开机自启
sudo systemctl enable --now nftables
```

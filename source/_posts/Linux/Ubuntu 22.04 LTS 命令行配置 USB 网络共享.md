---
title: Ubuntu 22.04 LTS 命令行配置 USB 网络共享
date:  2026-06-13 09:20:00

math: false
mermaid: false
category_bar: false

categories:
  - Linux

tags:
  - Network

excerpt: "使用手机 USB 网络共享给服务器联网"
permalink: /posts/20260613-092000.html
---
## 1. 环境

- Ubuntu 22.04 LTS

## 2. 启用 USB 网卡

USB 网络共享是指通过 USB 接口将手机虚拟为一个以太网卡，将移动网络共享给其他设备。需要在手机上开启 USB 网络共享设置，并使用数据线连接手机和服务器。

{% note warning %}
注：请确认数据线的线芯里真的有“数据线”，而不是一条廉价的充电线。如果不确认线材是否满足需求，推荐使用手机的原厂数据线。
{% endnote %}

### 2.1 查找虚拟网卡

使用 `ip link show` 来查看当前系统识别到的所有网络接口。在输出的内容中寻找以 `usb0` 或者 `enx`、`enp` 开头的网卡，在接下来的步骤中，请将 `usb0` 替换为实际查到的网卡名称。

### 2.2 启用网卡

使用以下命令启用 USB 网卡：

```Bash
sudo ip link set usb0 up
```

### 2.3 获取 IP 地址

运行 DHCP 客户端来自动获取 IP：

```Bash
sudo dhclient usb0
```

### 2.4 测试网络

首先使用 `ip a` 查看该网卡是否获取到 IP 地址，然后使用 `ping bing.com` 测试网络连通性。如果发现 ICMP 不可达，请参考以下内容。

## 3. 路由冲突

服务器本身会连接一个或者多个网络，使用 USB 共享网络时，不同网络的路由规则有概率发生冲突。如果系统把“访问外网”的任务分配给了内网网卡，就会导致网络流量无法正确通过手机虚拟的以太网卡。

### 3.1 断开内网连接

既然如此，最简单的办法就是把其它的网线拔掉，只保留手机 USB 共享的网络，系统就会自动将 USB 网络作为唯一的默认网关。

### 3.2 修改路由表

但是当需要保留内网连接时，就需要让系统优先走 USB 网卡。

使用以下命令查看路由表：

```Bash
ip route
```

如果发现 `default` 规则后边跟的网关是内网网关，就证实是默认网关走错了。

{% note danger %}
注：请勿直接删除默认路由！

如果你当前正通过外网、跨网段或 VPN 使用 SSH 远程连接服务器，执行此命令会切断服务器的返回路径，导致 SSH 瞬间卡死并彻底失联！
{% endnote %}

相比于粗暴地删除并重新添加，更优雅的做法是利用跃点数（Metric）来控制路由优先级。在 Linux 中，当存在多条 `default` 默认路由时，系统会优先选择 Metric 值**更小**的路径。

假设原有默认路由的 metric 为 100，只需添加一条 metric 更小的路由规则指向 USB 网卡即可。命令如下：

```Bash
# xxx.xxx.xxx.xxx 为手机为该 USB 共享网络分配的网关地址
sudo ip route add default via xxx.xxx.xxx.xxx dev usb0 metric 50
```

{% note success %}
再次运行 `ip route`，如果看到带有 `dev usb0 metric 50` 的默认路由排在原有默认路由的前面，说明配置成功。
{% endnote %}

## 4. DNS 服务器配置

如果尝试以上操作后，仍无法连通，请确认是否为 DNS 解析问题，命令如下：

```Bash
ping 8.8.8.8
# or
ping 114.114.114.114
```

如果可以 ping 通，说明是 DNS 服务器配置问题。

### 4.1 临时配置

通常使用 USB 网络共享均为临时操作，此时可以使用如下命令：

```Bash
# 设置 DNS 服务器
sudo resolvectl dns usb0 8.8.8.8 114.114.114.114

# 设置该网卡的 DNS 配置作为全局默认路由
sudo resolvectl domain usb0 "~."

# 验证 DNS 是否下发成功
resolvectl status usb0
```

### 4.2 永久配置

当在使用 4G USB 网卡时，可能需要永久配置。

Ubuntu 22.04 使用 Netplan 管理网络。进入 `/etc/netplan/` 目录，找到 `.yaml` 配置文件，加入 `usb0` 的配置：

```Yaml
network:
  ethernets:
    # ...... (保留原有配置)
    usb0:
      dhcp4: true
      nameservers:
        addresses: [8.8.8.8, 114.114.114.114]
  version: 2
```

然后应用配置：

```Bash
sudo netplan apply
```

{% note info %}
注：如果采用 Netplan 进行配置，Netplan 会在开机或网卡插入时自动接管 `usb0` 的启用和 IP 获取，无需再手动执行本文第 2 节的命令。
{% endnote %}
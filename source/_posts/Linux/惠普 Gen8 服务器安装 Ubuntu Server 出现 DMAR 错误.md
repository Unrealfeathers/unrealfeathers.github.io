---
title: 惠普 Gen8 服务器安装 Ubuntu Server 出现 DMAR 错误
date:  2026-03-14 17:07:38

math: false
mermaid: false
category_bar: false

categories:
  - Linux

tags:
  - Ubuntu Server
  - HP Gen8

excerpt: "DMAR: ERROR: DMA PTE for vPFN 0x428ee already set (to 3f6ee003 not 3f6ee003)"
permalink: /posts/20260314-170738.html
---
## 1. 环境

- HP ProLiant MicroServer Gen8
- Intel® Xeon® Processor E3-1265L V2
- DDR3 ECC 8G * 2
- Ubuntu Server 24.04.4 LTS

## 2. 报错

使用命令行用户登录后，系统会报出如下错误：

```Bash
 DMAR: ERROR: DMA PTE for vPFN 0x428ec already set (to 3f6ec003 not 3f6ec003)
 DMAR: ERROR: DMA PTE for vPFN 0x428ed already set (to 3f6ed003 not 3f6ed003)
 DMAR: ERROR: DMA PTE for vPFN 0x428ee already set (to 3f6ee003 not 3f6ee003)
 DMAR: ERROR: DMA PTE for vPFN 0x428ef already set (to 3f6ef003 not 3f6ef003)
 DMAR: ERROR: DMA PTE for vPFN 0x428f0 already set (to 3f6f0003 not 3f6f0003)
```

`DMAR (DMA Remapping)` 是 Intel VT-d（I/O 虚拟化技术，也叫 IOMMU）的一部分。它的作用是管理硬件设备（如网卡、显卡）直接访问内存（DMA）的权限。

而之所以报错，是因为 Linux 内核的 IOMMU 驱动在尝试映射一段内存地址给硬件时，发现这段地址已经被占用了。在 HP Gen8 这种老机器上，通常是因为主板集成的显卡（如 Matrox G200）或某些内置硬件在 BIOS 初始化阶段已经占用了这些 DMA 映射，而新版 Linux 内核的 IOMMU 校验变得更加严格，两者发生了冲突。

## 3. 解决办法

这里一共有两种解决办法，具体选择那种，取决于你的机器用途。

### 3.1 修改 Linux 内核引导参数

通过让 IOMMU 工作在 "Passthrough"（直通）模式，可以保留虚拟机的直通能力。不会影响你以后玩虚拟机（如 PVE/KVM）时的 PCI 硬件直通。

#### 3.1.1 编辑文件

```Bash
sudo vim /etc/default/grub
```

修改如下：

```Text
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash intel_iommu=pt"
```

{% note warning %}
可以将 `intel_iommu=pt` 换成 `intel_iommu=off`，效果同方法二，但不需要改 BIOS，可以方便一些；
{% endnote %}

#### 3.1.2 更新配置

```Bash
sudo update-grub
sudo reboot
```

### 3.2 关闭 VT-d (IOMMU)

如果你的 HP Gen8 只是单纯用来做 NAS，不需要运行需要**硬件直通**的虚拟机，你可以直接从硬件底层关掉它。

1. 重启服务器，在启动画面按 `F9` 进入 BIOS 设置；

2. 导航到 System Options -> Processor Options；

3. 找到 Intel (R) VT-d 或 Intel Virtualization Technology for Directed I/O；

4. 将其设置为 Disabled；

5. 按 `F10` 保存并重启机器；

---
title: 惠普 Gen8 使用 TF 卡引导系统启动
date:  2026-03-13 20:37:13

math: false
mermaid: false
category_bar: false

categories:
  - Linux

tags:
  - GRUB 2
  - HP Gen8

excerpt: "使用 TF 卡安装 Super GRUB2 Disk 引导系统启动"
permalink: /posts/20260313-203713.html
---
## 1. HP Gen8 介绍

HP ProLiant MicroServer Gen8，一款极为经典的 NAS 主机，有4个3.5寸盘位，一个 PCIE X16 长度插槽。特别是其支持 ILO 远程管理，不用接显示器就可以轻松安装系统。很多玩家都喜欢在前面的4个盘位塞满大容量机械硬盘作为数据盘，然后把系统安装在光驱位（SATA5）的固态硬盘上。

但这里有一个经典的问题：Gen8 的 BIOS 不支持直接从 SATA5 接口引导启动（尤其是在前面 1~4 槽位插了硬盘的情况下）。为了解决这个问题，最稳妥且优雅的方案就是使用主板自带的 TF 卡槽来作为引导跳板。

## 2. 制作引导用 TF 卡

> [Rufus - 轻松创建 USB 启动盘](https://rufus.ie)
> [Rescue your Windows & GNU/Linux systems](https://www.supergrubdisk.org/)

1. 插入准备好的 TF 卡；

2. 打开 Rufus 软件，设备选择你的 TF 卡，引导类型选择下载的 Super GRUB2 Disk 镜像；

3. 点击“开始”进行烧录，等待进度条走完，引导盘就初步做好了；

## 3. 修改 GRUB 配置文件

烧录完成后，先不要急着拔出 TF 卡。我们需要修改里面的配置文件，让它能够精准定位到 SATA5 上的系统。

在电脑上打开烧录好的 TF 卡，进入 `\boot\grub` 文件夹。找到 `grub.cfg` 文件，右键使用文本编辑器打开它。清空里面的原有内容，然后将以下代码复制粘贴进去：

```YAML
set timeout=3

menuentry 'NAS' {
  set root=(hd5)
  chainloader +1
}
```

{% note info %}
核心注意点，关于 `(hd5)` 的计算方法，这里的 `(hd5)` 并不是固定的，你需要根据当前 HP Gen8 前置 SATA1~4 盘位中实际装入的硬盘数量来动态调整：

- 计算公式：装入硬盘数量 + 1
{% endnote %}

## 4. 插卡开机

1. 保存修改好的 `grub.cfg` 文件，安全弹出 TF 卡。

2. 将卡插入 HP Gen8 主板上的 TF 卡槽中。

3. 开机启动，BIOS 会默认从 TF 卡读取引导信息，倒计时3秒后，就会顺滑地将接力棒交给 SATA5 上的系统。

---
title: GRUB 2 修改启动文件
date:  2026-03-15 21:07:43

math: false
mermaid: false
category_bar: false

categories:
  - Linux

tags:
  - GRUB 2

excerpt: "临时修改启动文件，进入系统"
permalink: /posts/20260315-210743.html
---
## 1. 临时修改启动文件

1. 在 GRUB 2 界面选择需要修改的选项；

2. 按 `E` 进入编辑模式，修改内容；

3. 修改完成后，按 `Ctrl + X` 或者 `F10` 保存并立即启动；

{% note danger %}
需注意，此为一次性操作，重启后失效。
{% endnote %}

## 2. 永久修改启动文件

进入 Linux 系统后，想要永久更改 GRUB 2 菜单，不要修改 `/boot/grub/grub.cfg`，按照以下步骤操作：

```Bash
# 编辑文件
sudo vim /etc/default/grub

# 应用修改
sudo update-grub
```
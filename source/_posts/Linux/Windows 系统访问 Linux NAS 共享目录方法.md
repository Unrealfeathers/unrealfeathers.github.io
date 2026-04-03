---
title: Windows 系统访问 NAS 共享目录方法
date:  2025-10-19 12:47:22

math: false
mermaid: false
category_bar: false

categories:
  - Linux

tags:
  - SSHFS

excerpt: "NAS 使用 SSHFS 共享目录"
permalink: /posts/20251019-124722.html
---
## 1. Windows 安装软件

### 1.1 安装 WinFsp

> [Releases · winfsp/winfsp](https://github.com/winfsp/winfsp/releases)

### 1.2 安装 SSHFS-Win

> [Releases · winfsp/sshfs-win](https://github.com/winfsp/sshfs-win/releases)

## 2. 映射驱动器

安装完成后，不需要打开任何独立的软件，它已经深度集成到了 Windows 资源管理器中，映射步骤如下：

1. 打开 Windows 的**文件资源管理器**；

2. 在左侧导航栏点击**此电脑**；

3. 在顶部菜单栏点击**映射网络驱动器**；

4. 选择一个未被使用的驱动器号；

5. 在**文件夹**路径框中，输入 `SSHFS-Win` 特有的路径格式。根据你的需求选择以下一种语法：

- 映射用户的 Home 目录 (`~`)：

  ```Text
  \\sshfs\用户名@IP
  ```

- 映射根目录 (`/`)：

  ```Text
  \\sshfs.r\用户名@IP
  ```

  {% note info %}
  `.r` 代表 `root`；
  {% endnote %}

- 映射特定的绝对路径（`/mnt`）：

  ```Text
  \\sshfs.r\用户名@IP\mnt
  ```

6. 勾选“登录时重新连接”，然后点击“完成”；

7. Windows 会弹出一个凭据输入框，输入 NAS 的用户名和密码；

8. 勾选“记住我的凭据”以便下次自动连接；

## 3. 注意事项

如果共享的目录为挂载目录，请调整权限，否则普通用户可能无法访问：

```Bash
# 将挂载目录的所有权改为当前用户
sudo chown -R $USER:$USER /mnt/disk
```

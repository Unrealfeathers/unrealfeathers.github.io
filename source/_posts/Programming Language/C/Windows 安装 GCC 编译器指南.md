---
title: Windows 安装 GCC 编译器指南
date:  2022-05-03 11:46:48

math: false
mermaid: false
category_bar: false

categories:
  - Programming Language

tags:
  - C
  - GCC
  - MSYS2

excerpt: "Windows 10/11 系统安装 GCC 编译器教程"
permalink: /posts/20220503-114648.html
---
## 1. 安装 MSYS2

> [MSYS2](https://www.msys2.org/)

MSYS2 是一个在 Windows 上提供类 Unix 环境的软件分发和构建平台，它包含了最新版本的 GCC。

1. 在官网首页向下滚动下载对应版本的安装程序并运行；

2. 安装完成后，在开始菜单中找到并打开 MSYS2 MSYS 终端；

3. 使用以下命令配置镜像源：

```Bash
sed -i "s#https\?://mirror.msys2.org/#https://mirrors.tuna.tsinghua.edu.cn/msys2/#g" /etc/pacman.d/mirrorlist*
```

4. 在终端中输入以下命令以更新包数据库和核心系统包，然后按回车：

```Bash
pacman -Syu
```

5. 如果提示关闭终端，请按 Y 确认关闭。然后再从开始菜单重新打开 MSYS2 MSYS 终端，输入以下命令更新剩余的包：

```Bash
pacman -Su
```

## 2. 安装 GCC

如今推荐使用基于 UCRT (Universal C Runtime) 的环境，它是更新、更符合现代 Windows 标准的 C 运行时库。

在 MSYS2 UCRT64 终端中，输入以下命令来安装包含 GCC、G++ 以及 GDB 调试器在内的完整工具链：

```Bash
pacman -S mingw-w64-ucrt-x86_64-toolchain
```

## 3. 配置环境变量

为了能在 Windows 的 CMD 或 VS Code 等编辑器中直接使用 `gcc`，需要将其添加到系统环境变量中。

1. 在 Windows 搜索栏中输入“环境变量”，选择“编辑系统环境变量”；

2. 在弹出的窗口中，点击右下角的“环境变量”按钮；
    
3. 在下方的“系统变量”区域中，找到名为 `Path` 的变量，选中它并点击“编辑”；

4. 点击“新建”，然后输入 MSYS2 中 GCC 的安装路径。如果是默认安装，请输入：

```Text
C:\msys64\ucrt64\bin
```

5. 一路点击“确定”保存设置；

## 4. 验证安装

打开终端，输入以下命令：

```Bash
gcc --version
g++ --version
gdb --version
```

如果输出了对应的版本信息，说明安装成功。
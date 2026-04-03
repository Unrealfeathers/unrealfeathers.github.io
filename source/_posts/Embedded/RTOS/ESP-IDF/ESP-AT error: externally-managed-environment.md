---
title: "ESP-AT error: externally-managed-environment"
date:  2026-03-21 09:19:39

math: false
mermaid: false
category_bar: false

categories:
  - Embedded System

tags:
  - FreeRTOS
  - ESP-IDF
  - ESP-AT

excerpt: "error: externally-managed-environment"
permalink: /posts/20260321-091939.html
---
## 1. 报错信息

```Text
Ready to install ESP-AT prerequisites..
error: externally-managed-environment

× This environment is externally managed
╰─> To install Python packages system-wide, try apt install
    python3-xyz, where xyz is the package you are trying to
    install.
    
    If you wish to install a non-Debian-packaged Python package,
    create a virtual environment using python3 -m venv path/to/venv.
    Then use path/to/venv/bin/python and path/to/venv/bin/pip. Make
    sure you have python3-full installed.
    
    If you wish to install a non-Debian packaged Python application,
    it may be easiest to use pipx install xyz, which will manage a
    virtual environment for you. Make sure you have pipx installed.
    
    See /usr/share/doc/python3.12/README.venv for more information.

note: If you believe this is a mistake, please contact your Python installation or OS distribution provider. You can override this, at the risk of breaking your Python installation or OS, by passing --break-system-packages.
hint: See PEP 668 for the detailed specification.
A fatal error occurred: install ESP-AT prerequisites failed!
```

## 2. 解决办法

这个错误的根本原因是较新的发行版启用了 `PEP 668` 保护机制。为了防止用户通过 pip 全局安装的 Python 包与系统自带的包管理器（如 apt）产生冲突并导致系统崩溃，较新的 Linux 系统禁止了 pip 在全局环境中直接安装包。

ESP-AT 的安装脚本在底层尝试使用 `pip install` 为你全局安装必要的 Python 依赖（Prerequisites），因此被操作系统直接拦截并报错：externally-managed-environment（外部管理的系统环境）。

### 2.1 安装虚拟环境工具

```Bash
sudo apt update
sudo apt install python3-venv python3-full
```

### 2.2 创建虚拟环境

```Bash
cd ~/esp/esp-at
python3 -m venv .venv
```

### 2.3 激活虚拟环境

```Bash
# 进入环境
source .venv/bin/activate

# 退出环境
deactivate
```

### 2.4 重新运行安装脚本

```Bash
./build.py install
```

下次开发 ESP-AT 时，记得使用 `source .venv/bin/activate` 来激活环境，然后再运行编译命令；
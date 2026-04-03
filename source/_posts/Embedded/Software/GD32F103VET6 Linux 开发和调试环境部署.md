---
title: GD32F103VET6 Linux 开发和调试环境部署
date:  2026-03-29 23:08:57

math: false
mermaid: false
category_bar: false

categories:
  - Embedded System

tags:
  - C
  - GDB
  - GD32
  - VS Code
  - OpenOCD

excerpt: "ARM 交叉编译链、GDB、OpenOCD、DAP-Link"
permalink: /posts/20260329-230857.html
---
## 1. Windows 环境搭建

- Windows 10 22H2

### 1.1 VS Code

> [Visual Studio Code - The open source AI code editor](https://code.visualstudio.com/)

#### 1.1.1 安装

1. 安装 VS Code；

2. 配置基础的 `C/C++` 环境；

3. 安装 `Remote - SSH` 插件；

4. 新建远程，使用 VS Code 连接到远程服务器；

#### 1.1.2 配置 SSH 端口反代

为了实现远程调试，需要让云服务器的 3333 端口流量，通过 SSH 隧道转发到本地 Windows 的 3333 端口。结合 VS Code 和 `Remote-SSH`，最优雅的方式是修改 Windows 的 SSH 配置文件。

1. 在本地 Windows 上，打开 `C:\Users\你的用户名\.ssh\config` 文件；

2. 在你的云服务器配置节点下，添加 `RemoteForward` 指令：

```TEXT
Host Ubuntu-Cloud
    HostName <服务器公网IP>
    User <用户名>
    Port 22
	# 配置反向转发，将服务端 3333 端口收到的请求转发至 localhost:3333
    RemoteForward 3333 127.0.0.1:3333
```

### 1.2 Open OCD

> [Releases · xpack-dev-tools/openocd-xpack](https://github.com/xpack-dev-tools/openocd-xpack/releases)

#### 1.2.1 安装

1. 下载 Windows 版本的 `.zip` 包；

2. 解压到如 `C:\Development\openocd`；

3. 将 `bin` 文件夹路径添加到系统的 `Path` 环境变量中；

4. 打开 CMD 输入 `openocd --version` 验证；

#### 1.2.2 使用

Open OCD 启动时需要两个核心配置文件：**接口脚本（Interface）和目标芯片脚本（Target）**；我使用的是 **DAP-Link**，先将 **DAP-Link** 连接到 Windows 电脑，在终端输入如下命令：

```Shell
# GD32F103 的内核和调试逻辑与 STM32F103 基本兼容，直接使用 `target/stm32f1x.cfg` 通常是可以直接工作的；
# -f 后面接的是调试器配置文件，-f 后面接的是芯片配置文件；
openocd -f interface/cmsis-dap.cfg -f target/stm32f1x.cfg
```

最后一行输出如下，启动成功：

```Text
Info : Listening on port 3333 for gdb connections
```

## 2. Linux 环境搭建

- Ubuntu Server 24.04 LTS

### 2.1 安装交叉编译工具链和调试器

```Shell
sudo apt update
sudo apt install git make cmake build-essential -y
sudo apt install gcc-arm-none-eabi gdb-multiarch binutils-arm-none-eabi -y
```

### 2.2 安装 VS Code 插件

在云服务器上，通过 VS Code 的插件市场安装插件（注意要安装在 `SSH: Remote` 这一侧），插件列表如下：

- C/C++

- C/C++ DevTools

- C/C++ Extension Pack

- C/C++ Themes

- Chinese (Simplified) (简体中文) Language Pack for Visual Studio Code

- CMake Tools

- Cortex-Debug

## 3 工程配置

> [Unrealfeathers/hello_led at bare_metal](https://github.com/Unrealfeathers/hello_led/tree/bare_metal)

目录结构如下：

```Text
hello_led/
├── .vscode/                    # VS Code 配置文件
│   ├── c_cpp_properties.json
│   ├── launch.json
│   ├── settings.json
│   └── tasks.json
├── app/                        # 应用层文件
│   ├── inc/
│   │   ├── gd32f10x_it.h
│   │   ├── gd32f10x_libopt.h
│   │   └── sys_tick.h
│   ├── src/
│   │   ├── gd32f10x_it.c
│   │   ├── sys_calls.c
│   │   └── sys_tick.c
│   └── main.c
├── build/                      # 编译输出目录 (由 CMake 自动生成)
├── lib/                        # 第三方库 / 厂商固件库
│   ├── CMSIS/                  # ARM Cortex 核心文件及 GD32 启动代码/系统文件
│   │   └── GD/
│   │       └── GD32F10x/
│   │           ├── Include/
│   │           └── Source/
│   └── GD32F10x_standard_peripheral/ # GD32 标准外设库
│       ├── Include/
│       └── Source/
├── startup/                    # 汇编启动文件和链接脚本
│   ├── gd32f10x_flash.ld
│   └── startup_gd32f10x_hd.S
├── svd/                        # 存放外设寄存器描述文件
│   └── GD32F10x_HD.svd
├── .gitignore                  # Git 版本控制
├── CMakeLists.txt              # CMake 构建脚本
└── toolchain.cmake             # 交叉编译工具链配置
```

### 3.1 VS Code 工程与调试配置

#### 3.1.1 `c_cpp_properties.json`

C/C++ 开发环境配置，注意需要使用 `arm-none-eabi-gcc` ARM 交叉编译工具链：

```Json
{
    "configurations": [
        {
            "name": "Linux",
            "includePath": [
                "${workspaceFolder}/**"
            ],
            "defines": [],
            "cStandard": "c23",
            "cppStandard": "gnu++23",
            "intelliSenseMode": "linux-gcc-x64",
            "compilerPath": "/usr/bin/arm-none-eabi-gcc"
        }
    ],
    "version": 4
}
```

#### 3.1.2 `launch.json`

VS Code 调试配置：

```Json
{
    "version": "1.0.0",
    "configurations": [
        {
            "name": "GD32 Remote Debug",
            "type": "cortex-debug",
            "request": "launch",
            "servertype": "external",       // 核心：告诉插件使用外部 GDB Server，不要启动 OpenOCD
            "gdbTarget": "localhost:3333",  // 核心：指向通过 SSH 反代过来的本地 OpenOCD 端口
            "cwd": "${workspaceFolder}",
            "executable": "${workspaceFolder}/build/hello_led.elf", // 替换为你的 .elf 文件路径
            "gdbPath": "/usr/bin/gdb-multiarch", // 云端安装的多架构 GDB 路径
            "runToEntryPoint": "main", // 烧录后自动运行到 main 函数并停住
            "showDevDebugOutput": "none",
            "svdFile": "${workspaceFolder}/svd/GD32F10x_HD.svd", // 强烈推荐：指向你的 GD32 SVD 文件路径
            "toolchainPrefix": "arm-none-eabi-",      // 指定工具链前缀
            "objdumpPath": "arm-none-eabi-objdump",   // 明确指定 objdump
            "preLaunchTask": "build" // 可选：绑定你的编译任务
        }
    ]
}
```

{% note info %}
支持 `.svd` 文件是使用 `Cortex-Debug` 插件的最大优势。有了 `.svd` 文件，就可以在调试时直接查看和修改 GD32 的所有外设寄存器（如 GPIO、Timer、ADC 等）。
{% endnote %}

#### 3.1.3 `settings.json`

设置 CMake 路径：

```Json
{
    "cmake.sourceDirectory": "/home/ubuntu/GD32F10x_FreeRTOS/User/hello_led",
    "cortex-debug.variableUseNaturalFormat": false
}
```

#### 3.1.4 `tasks.json`

创建编译任务：

```Json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "cmake_config",
            "type": "shell",
            "command": "cmake -E rm -rf build && cmake -B build -DCMAKE_TOOLCHAIN_FILE=toolchain.cmake",
            "problemMatcher": []
        },
        {
            "label": "build",
            "type": "shell",
            "command": "cmake --build build -j4",
            "dependsOn": [
                "cmake_config"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "problemMatcher": ["$gcc"],
        }
    ]
}
```

### 3.2 工程代码

请参考我的 GitHub 仓库。

## 4. 编译调试流程

整个架构的数据流向如下：

**VS Code (UI) -> Remote-SSH -> 云端 Ubuntu (GDB) -> SSH 反向隧道 -> 本地 Windows (OpenOCD) -> 烧录器 (DAP-Link) -> GD32 芯片**

### 4.1 手动编译流程

```Shell
# 1. 生成 Makefile
cmake -B build -DCMAKE_TOOLCHAIN_FILE=toolchain.cmake

# 2. 执行编译 (使用 4 个线程加速编译)
cmake --build build -j4
```

### 4.2 自动编译调试

1. **本地 Windows**：保持插好烧录器，并在命令行启动 OpenOCD，监听本地的 3333 端口；

2. **VS Code**：由于配置了 SSH 的 `RemoteForward 3333 127.0.0.1:3333`，云端的 3333 端口此时已经与本地连通；

3. **一键调试**：在 VS Code 中按下 `F5`；

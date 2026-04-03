---
title: ESP-AT 开发环境部署
date:  2026-03-22 11:40:41

math: false
mermaid: false
category_bar: false

categories:
  - Embedded System

tags:
  - FreeRTOS
  - ESP-IDF
  - ESP-AT

excerpt: "在云服务器上部署 ESP-AT 开发环境"
permalink: /posts/20260322-114041.html
---
> [本地编译 ESP-AT 工程 - ESP32-C6 - — ESP-AT 用户指南 release-v5.0.0.0 文档](https://docs.espressif.com/projects/esp-at/zh_CN/release-v5.0.0.0/esp32c6/Compile_and_Develop/How_to_clone_project_and_compile_it.html)

## 1. 系统环境

- Ubuntu Server 24.04 LTS
- ESP32-C6-DevKitM-1 开发板

## 2. 获取 ESP-AT

```Bash
# 创建目录
mkdir esp
cd ~/esp

# 克隆工程
git clone --recursive https://github.com/espressif/esp-at.git
```

### 2.1 切换特定版本

可以自己选择对应的 ESP-AT 版本，我这里选择了 ESP32-C6 最推荐的 V4.1.1.0：

```Bash
# 进入工程目录
cd ./esp-at

# 拉取最新的远程代码和标签
git fetch --all --tags

# 检出特定标签
git checkout v4.1.1.0
```

## 3. 安装环境

### 3.1 安装虚拟环境工具

```Bash
sudo apt update
sudo apt install python3-venv python3-full python-is-python3
```

### 3.2 创建虚拟环境

```Bash
cd ~/esp/esp-at
python3 -m venv .venv
```

### 3.3 激活虚拟环境

```Bash
# 进入虚拟环境
source .venv/bin/activate

# 退出虚拟环境
deactivate
```

### 3.4 安装 ESP-AT 软件包

```Bash
# 进入虚拟环境
cd ~/esp/esp-at
source .venv/bin/activate

# 安装依赖
./build.py install
```

请根据自己的芯片进行选择，ESP32-C6 的配置如下：

```Text
Platform name:
1. PLATFORM_ESP32
2. PLATFORM_ESP32C3
3. PLATFORM_ESP32C2
4. PLATFORM_ESP32C5
5. PLATFORM_ESP32C6
6. PLATFORM_ESP32S2
choose(range[1,6]):5

Module name:
1. ESP32C6-4MB  (Firmware description: 4MB, Wi-Fi + BLE, OTA, TX:7 RX:6)
choose(range[1,1]):1

Enable silence mode to remove some logs and reduce the firmware size?
0. No
1. Yes
choose(range[0,1]):1
Platform name:ESP32C6   Module name:ESP32C6-4MB Silence:1
```

### 3.5 安装 ESP-IDF 软件包

在安装完成 ESP-AT 的软件包后，会自动安装 ESP-IDF 的软件包。因为 ESP-IDF 的安装脚本会自动创建一个虚拟环境，此时会报错：

```Text
Ready to set up ESP-IDF tools..
ERROR: This script was called from a virtual environment, can not create a virtual environment again
A fatal error occurred: set up ESP-IDF python-env failed
```

报错原因是安装 ESP-IDF 的软件包时发生了虚拟环境的嵌套。想要解决也很简单，只需要进入到 `ESP-IDF` 目录下手动执行安装脚本就好了。步骤如下：

```Bash
# 退出虚拟环境
deactivate

# 执行安装脚本
cd ./esp-idf
./install.sh
```

## 4. （可选）自定义 AT 端口管脚

默认情况下，ESP-AT 使用两个 UART 接口作为 AT 端口：一个用于输出日志（以下称为日志端口），另一个用于发送 AT 命令和接收响应（以下称为命令端口）。

### 4.1 修改日志端口管脚

```Bash
./build.py menuconfig
```

配置项路径：

`Component config` -> `ESP System Settings` -> `Channel for console output`

### 4.2 修改命令端口管脚

默认情况下，`UART1` 用于发送 AT 命令和接收 AT 响应，其管脚定义在 [factory_param_data.csv](https://github.com/espressif/esp-at/blob/583d2096/components/customized_partitions/raw_data/factory_param/factory_param_data.csv) 表格中的 `uart_port、uart_tx_pin、uart_rx_pin、uart_cts_pin` 和 `uart_rts_pin` 列。

## 5. 编译和烧录

记得使用 `source .venv/bin/activate` 来激活环境，然后再运行编译命令；

### 5.1 编译

```Bash
./build.py build
```

### 5.2 烧录

```Bash
./build.py flash -p /dev/端口号
```

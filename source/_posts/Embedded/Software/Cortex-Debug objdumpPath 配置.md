---
title: Cortex-Debug objdumpPath 配置
date:  2026-04-02 19:10:51

math: false
mermaid: false
category_bar: false

categories:
  - Embedded System

tags:
  - Cortex-Debug
  - GD32

excerpt: "使用 Cortex-Debug 插件调试 GD32 报错无法找到 objdump 和 nm"
permalink: /posts/20260402-191051.html
---
## 1. 报错信息

在使用 Cortex-Debug 和 ARM 交叉编译工具链进行调试时，报错如下：

```Text
Error: /usr/bin/nm-multiarch failed! statics/global/functions may not be properly classified: Error: spawn /usr/bin/nm-multiarch ENOENT

Expecting `nm` next to `objdump`. If that is not the problem please report this.

Error: objdump failed! statics/globals/functions may not be properly classified: Error: spawn /usr/bin/objdump-multiarch ENOENT

ENOENT means program not found. If that is not the issue, please report this problem.
```

报错原因：

Cortex-Debug 插件在启动时，会尝试调用 `objdump` 和 `nm` 这两个工具来解析 `.elf` 文件，以便在 VS Code 左侧的面板中为你显示全局变量、静态变量和函数列表。它默认去寻找了 Ubuntu 系统级的多架构版本（/usr/bin/objdump-multiarch），但系统没有安装这个包。此外，应该让它使用专门针对 ARM 交叉编译的工具链。

## 2. 解决办法

为了让 Cortex-Debug 去使用 `arm-none-eabi` 工具链来解析变量和函数名，在 `launch.json` 中添加如下两行：

```Json
{
    "configurations": [
        {
            // ......
            "toolchainPrefix": "arm-none-eabi-",      // 指定工具链前缀
            "objdumpPath": "arm-none-eabi-objdump",   // 明确指定 objdump
            // ......
        }
    ]
}
```

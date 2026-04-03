---
title: Keil 调试卡在 BKPT 0xAB 断点原因
date:  2026-03-31 22:35:13

math: false
mermaid: false
category_bar: false

categories:
  - Embedded System

tags:
  - RTOS
  - RT-Thread

excerpt: "Keil 调试时，卡在 0x08042AC4 BEAB BKPT 0xAB 断点"
permalink: /posts/20260331-223513.html
---
## 1. 原因

Keil 中看到的 `BKPT 0xAB`（Breakpoint 0xAB）是 ARM 架构中极其经典的指令，它代表系统触发了半主机模式。

当代码中调用了标准 C 库函数（例如：`printf`），但没有完全重定向到物理串口。在没有连接调试器，进行冷启动时，CPU 执行到这里就会直接卡死。连上调试器后，调试器接管了这个指令，触发了半主机模式。

半主机模式是 ARM 的一种机制：当单片机没有屏幕和键盘时，如果在代码里调用了 `printf` 打印信息，或者因为发生错误触发了 C 库的 `assert、exit()`，C 库底层不知道该把数据发往哪里，就会执行一条软中断指令 `BKPT 0xAB`。这条指令的意思是：“呼叫调试器！请把我这里的数据通过仿真器显示到电脑屏幕上！”

- 冷启动（拔掉仿真器，纯上电）：系统遇到 `BKPT 0xAB`，因为没有调试器响应，CPU 直接抛出硬件错误或者死循环，系统直接挂掉；

- 热启动/调试模式（插着仿真器）：系统遇到 `BKPT 0xAB`，会被 Keil 捕获，这就是为什么会看到执行指示器的黄色箭头精准地卡在这一行的原因；

## 2. 解决办法

代码中搜索一下，看看是不是调用了标准 C 库函数（例如：`printf`），如果是，就将其删掉再尝试编译烧录和调试。

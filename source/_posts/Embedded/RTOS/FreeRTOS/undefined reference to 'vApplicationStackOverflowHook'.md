---
title: "undefined reference to 'vApplicationStackOverflowHook'"
date:  2026-04-01 18:47:05

math: false
mermaid: false
category_bar: false

categories:
  - Embedded System

tags:
  - RTOS
  - FreeRTOS

excerpt: "缺少 FreeRTOS 栈溢出钩子函数"
permalink: /posts/20260401-184705.html
---
## 1. 报错信息

```Text
undefined reference to 'vApplicationStackOverflowHook'
```

报错原因：

在 `FreeRTOSConfig.h` 中开启了栈溢出检查（`configCHECK_FOR_STACK_OVERFLOW` 被设置为了 1 或 2），当开启此功能时，FreeRTOS 要求用户必须自己实现一个名为 `vApplicationStackOverflowHook` 的回调函数，以便在发生栈溢出时系统知道该怎么处理。

## 2. 解决办法

### 2.1 提供钩子函数的实现

在调试阶段，推荐提供钩子函数的实现。当某个任务爆栈时，它会卡死在这里方便你用 Cortex-Debug 查看。

```C
void vApplicationStackOverflowHook(TaskHandle_t xTask, char *pcTaskName) {
    // 发生栈溢出时，进入死循环
    while (1) {
        // 可以在这里打个断点，或者输出日志
    }
}
```

### 2.2 关闭栈溢出检查

打开 `FreeRTOSConfig.h`，找到 `configCHECK_FOR_STACK_OVERFLOW` 宏，将其修改为 0。

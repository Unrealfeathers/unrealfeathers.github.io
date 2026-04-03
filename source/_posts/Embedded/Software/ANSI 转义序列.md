---
title: 串口如何打印彩色日志
date:  2022-10-18 18:36:09

math: false
mermaid: false
category_bar: false

categories:
  - Embedded System

tags:
  - 串口调试

excerpt: "在串口调试中，可以使用 ANSI 转义序列来控制光标位置、颜色、字体样式（如粗体、下划线）"
permalink: /posts/20221018-183609.html
---
## 1. ANSI 转义序列

在串口调试中，可以使用 ANSI 转义序列（ANSI Escape Sequences）来控制光标位置、颜色、字体样式（如粗体、下划线）。

### 1.1 序列结构

绝大多数控制序列都以 CSI（Control Sequence Introducer）开头：

1. `ESC` 字符：ASCII 码为 `27`（十六进制：`0x1B`，八进制：`\033`）；

2. 中括号 `[`：紧跟在 `ESC` 之后；

3. 通用格式：`\033[参数m`；

### 1.2 颜色代码表

想要打印出彩色的日志，可以参照 `\033[显示方式;前景色;背景色m` 格式。

#### 1.2.1 显示方式

| 显示方             | 代码 |
| ------------------ | --- |
| 终端默认设置（重置） | 0   |
| 高亮 / 加粗         | 1   |
| 下划线              | 4   |
| 闪烁                | 5   |

#### 1.2.2 颜色代码

| 颜色   | 前景色（Foreground） | 背景色（Background） |
| ----- | --------------- | --------------- |
| 黑色  | 30              | 40              |
| 红色  | 31              | 41              |
| 绿色  | 32              | 42              |
| 黄色  | 33              | 43              |
| 蓝色  | 34              | 44              |
| 洋红  | 35              | 45              |
| 青色  | 36              | 46              |
| 白色  | 37              | 47              |

## 2. 串口打印示例

代码示例：

```C
#include <stdio.h>

int main()
{
    printf("\033[1;31m ERROR: Message; \033[0m\r\n");
    printf("\033[1;33m WARN:  Message; \033[0m\r\n");
    printf("\033[1;32m INFO:  Message; \033[0m\r\n");

    return 0;
}
```

MobaXterm 显示结果：

```Text
 ERROR: Message; (显示为红色)
 WARN:  Message; (显示为黄色)
 INFO:  Message; (显示为绿色)
```

## 3. 注意事项

串口工具（例如：MobaXterm）必须要支持 ANSI 协议才能实现彩色打印。
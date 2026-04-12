---
title: Zlog 打印彩色日志
date:  2026-04-10 19:13:07

math: false
mermaid: false
category_bar: false

categories:
  - Linux

tags:
  - C
  - Zlog

excerpt: "使用 Zlog 实现仿 ESP-IDF 的彩色日志打印"
permalink: /posts/20260410-191307.html
---
## 1. 彩色日志打印

> [串口如何打印彩色日志 | Unrealfeathers' Blog](https://flowerdown.org/posts/20221018-183609)

通常情况下，我们可以使用 ANSI 转义序列（如 `\033[31m`）实现彩色的日志打印。但由于 Zlog 的配置解析器不支持处理 `\033` 这种转义语法，它会将 `\033` 这四个字符视作普通的字符串字面量直接输出，导致日志变成 `\033[1;31m ERROR: Message; \033[0m` 的形式。

所以直接在 `zlog.conf` 中写入如下格式是不行的：

```Ini
color_F = "\033[35m [%d(%Y-%m-%d %H:%M:%S)] [%V] [%c] [%f:%L]: %m \033[0m %n"
```

## 2. 解决办法

既然解析器不支持转义，那我们就绕过解析，直接把 **真实的 ESC 控制字符的 ASCII 码** 写入到配置文件中。在 Linux 终端下，可以使用如下 `echo -e` 命令来实现：

```Bash
echo -e 'color_F = "\033[35m [%d(%Y-%m-%d %H:%M:%S)] [%V] [%c] [%f:%L]: %m \033[0m %n"' > zlog.conf
```

这样，我们就通过 `echo` 将 ESC 这个特殊的不可见字符注入到配置中去了，后面如果需要新增配置项，直接复制粘贴就可以了。
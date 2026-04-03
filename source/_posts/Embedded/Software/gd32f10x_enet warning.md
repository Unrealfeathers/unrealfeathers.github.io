---
title: "gd32f10x_enet.h: 'enet_default_init' declared 'static' but never defined [-Wunused-function]"
date:  2026-03-27 18:46:10

math: false
mermaid: false
category_bar: false

categories:
  - Embedded System

tags:
  - C
  - GD32

excerpt: "'enet_default_init' declared 'static' but never defined [-Wunused-function] 'enet_delay' declared 'static' but never defined [-Wunused-function]"
permalink: /posts/20260327-184610.html
---
## 1. 报错信息

```Text
/../GD32F10x_standard_peripheral/Include/gd32f10x_enet.h: 'enet_default_init' declared 'static' but never defined [-Wunused-function]
/../GD32F10x_standard_peripheral/Include/gd32f10x_enet.h: 'enet_delay' declared 'static' but never defined [-Wunused-function]
```

警告原因：

GD32 的官方库希望把 `enet_default_init()` 和 `enet_delay()` 留给用户在自己的应用层代码中实现（例如用于以太网的初始化和延时）。但官方在 `gd32f10x_enet.h` 中错误地将它们声明为了 `static`，导致任何包含了这个头文件但没有在同一个文件里定义这两个函数的 `.c` 文件，都会触发 `[-Wunused-function]`（声明了 `static` 函数但未定义/未使用）的警告。

## 2. 解决办法

这两个函数本质上是供外部实现的全局函数，应该使用 `extern` 声明，或者干脆去掉 `static`（C 语言函数声明默认就是 `extern`）。

所以只需要删除 `static` 关键字，或者将其替换为 `extern`，保存文件后重新编译，警告就会彻底消失。如下：

```C
extern void enet_default_init(void);
extern void enet_delay(uint32_t ncount);
```

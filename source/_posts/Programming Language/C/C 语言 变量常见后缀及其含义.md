---
title: C 语言 变量常见后缀及其含义
date:  2025-08-23 17:42:39

math: false
mermaid: false
category_bar: false

categories:
  - Programming Language

tags:
  - C

excerpt: "C 语言中常见的变量后缀"
permalink: /posts/20250823-174239.html
---
## 1. 字面量后缀 (Literal Suffixes)

这是 C 语言语法的一部分。当写下一个数字（例如：10）时，编译器默认将其视为 `int`。如果需要强制指定类型，就需要用到后缀。

| 后缀    | 类型            | 示例     | 含义      |
| ------- | --------------- | ------- | ------- |
| `u/U`   | `unsigned int`  | `100u`  | 无符号整数   |
| `l/L`   | `long`          | `100L`  | 长整型     |
| `ll/LL` | `long long`     | `100LL` | 64 位长整型 |
| `ul/UL` | `unsigned long` | `100UL` | 无符号长整型  |
| `f/F`   | `float`         | `3.14f` | 单精度浮点数  |
| `l/L`   | `long double`   | `3.14L` | 扩展精度浮点数 |

{% note warning %}
注意：建议统一使用大写（如 `10L` 而不是 `10l`），因为小写的 `l` 在很多字体中极易与数字 `1` 混淆；
{% endnote %}

## 2. 命名习惯后缀 (Naming Conventions)

这些后缀不是语法强制要求的，但在工程实践中非常常见，用于标识变量的用途。

### 2.1 数据类型类

| 后缀   | 缩写  | 原型 | 类型    |
| ------ | ----- | ------- | ------- |
| `_ptr` | `_p`  | Pointer | 指针类型  |
| `_arr` | `_a`  | Array   | 数组类型  |
| `_t`   | `_t`  | Typedef | 自定义类型 |

### 2.2 业务逻辑类

| 后缀     | 缩写   | 原型    | 类型  |
| -------- | ------ | ------- | --- |
| `_count` | `_cnt` | Count   | 计数器 |
| `_idx`   | `_i`   | Index   | 索引  |
| `_buf`   | `_buf` | Buffer  | 缓冲区 |
| `_str`   | `_str` | String  | 字符串 |
| `_min`   | `_min` | Minimum | 最小值 |
| `_max`   | `_max` | Maximum | 最大值 |

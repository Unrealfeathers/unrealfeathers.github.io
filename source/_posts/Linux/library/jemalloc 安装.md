---
title: jemalloc 安装
date:  2026-04-08 18:44:14

math: false
mermaid: false
category_bar: false

categories:
  - Linux

tags:
  - jemalloc
  - CMake

excerpt: "Jemalloc 是一个由 Jason Evans 开发的通用 C/C++ 内存分配器"
permalink: /posts/20260408-184414.html
---
## 1. 安装

> [jemalloc/jemalloc](https://github.com/jemalloc/jemalloc)

环境如下：

- Ubuntu Server 24.04 LTS

安装步骤如下：

```Bash
sudo apt install libjemalloc-dev
```

## 2. 将 jemalloc 集成到 CMake

```CMake
# 查找 jemalloc 库文件
find_library(JEMALLOC_LIB jemalloc)

if(NOT JEMALLOC_LIB)
    message(FATAL_ERROR "Could not find jemalloc library. Please ensure jemalloc is installed.")
else()
    message(STATUS "Found jemalloc library at: ${JEMALLOC_LIB}")
endif()

# 链接三方库
target_link_libraries(${PROJECT_NAME} PRIVATE 
    ${JEMALLOC_LIB}
)
```

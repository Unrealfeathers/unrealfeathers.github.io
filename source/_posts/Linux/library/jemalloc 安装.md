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
# 查找 jemalloc 静态库
find_library(JEMALLOC_LIBRARY NAMES libjemalloc.a NAMES_PER_DIR)

if(JEMALLOC_LIBRARY)
    message(STATUS "Found static jemalloc: ${JEMALLOC_LIBRARY}")
else()
    message(WARNING "Static jemalloc (libjemalloc.a) not found! Will fall back to standard libc allocator.")
endif()

# 条件链接静态 jemalloc
if(JEMALLOC_LIBRARY AND CMAKE_BUILD_TYPE STREQUAL "Release")
    message(STATUS "Statically linking jemalloc for performance optimization.")

    # 链接静态库
    target_link_libraries(${PROJECT_NAME} PRIVATE ${JEMALLOC_LIBRARY})

    # 静态链接 jemalloc 通常需要显式链接底层依赖的 dl 和 pthread 库
    target_link_libraries(${PROJECT_NAME} PRIVATE ${CMAKE_DL_LIBS})
elseif(JEMALLOC_LIBRARY AND CMAKE_BUILD_TYPE STREQUAL "Debug")
    message(STATUS "Skipping jemalloc link in Debug mode to avoid conflict with AddressSanitizer/Valgrind.")
endif()
```

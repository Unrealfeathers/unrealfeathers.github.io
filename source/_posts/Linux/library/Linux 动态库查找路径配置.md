---
title: Linux 动态库查找路径配置
date:  2026-05-13 21:23:31

math: false
mermaid: false
category_bar: false

categories:
  - Linux

tags:
  - CMake

excerpt: "Linux 默认不会在程序所在的目录查找动态链接库"
permalink: /posts/20260513-212331.html
---
## 1. RPATH 机制

RPATH(Run-time search path) 可以硬编码到可执行文件中，告诉程序在运行时优先去哪里寻找动态库。

## 2. 设置 RPATH

在 `CMakeLists.txt` 设置 RPATH，告诉程序运行时优先在当前目录 `$ORIGIN` 及其 `libs` 子目录查找动态库：

```CMake
set_target_properties(${PROJECT_NAME} PROPERTIES
    BUILD_RPATH "$ORIGIN;$ORIGIN/libs"
    INSTALL_RPATH "$ORIGIN;$ORIGIN/libs"
)
```

{% note info %}
`${PROJECT_NAME}` 为项目名称
{% endnote %}

## 3. 设置自动拷贝

如果你希望每次编译后，CMake 自动把库文件拷贝到执行目录下，可以添加如下命令：

```CMake
add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD 
    COMMAND ${CMAKE_COMMAND} -E copy_directory 
    ${CMAKE_SOURCE_DIR}/libs 
    ${CMAKE_CURRENT_BINARY_DIR}/libs
    COMMENT "Copying dynamic libraries to output directory..."
)
```

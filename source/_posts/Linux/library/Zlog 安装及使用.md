---
title: Zlog 安装及使用
date:  2026-04-06 18:32:47

math: false
mermaid: false
category_bar: false

categories:
  - Linux

tags:
  - C
  - Zlog
  - CMake

excerpt: "Zlog 是一个可靠、高性能、线程安全、灵活、纯 C 语言的日志库"
permalink: /posts/20260406-183247.html
---
## 1. 安装

> [HardySimpson/zlog](https://github.com/HardySimpson/zlog/tree/master)

环境如下：

- Ubuntu Server 24.04 LTS

安装步骤如下：

```Bash
# 克隆仓库
git clone https://github.com/HardySimpson/zlog.git

# 编译
cd zlog
make

# 安装
sudo make installl

# 刷新系统动态链接库缓存
sudo ldconfig
```

## 2. 配置

在项目根目录下创建一个名为 `zlog.conf` 的文件。这个文件用来告诉 zlog 你的日志规则。内容如下：

```Ini
# [formats] 定义日志的输出格式
[formats]
# %d: 时间 | %-5V: 日志级别(左对齐占据5个字符) | %f: 文件名 | %-3L: 行号(左对齐占据3个字符) | %m: 日志信息 | %n: 换行
log_format = "[%d(%Y-%m-%d %H:%M:%S)] [%-5V] [%f:%-3L]: %m%n"

# [rules] 定义日志的路由规则 (分类名.日志级别)
[rules]
# 把 "project" 分类下所有的日志 (.*) 打印到标准输出
project.* >stdout; log_format

# 把 "project" 分类下 ERROR 及以上级别的日志 (包含 FATAL) 写到文件
# 当 error.log 达到 10MB 时自动回滚，保留最近的 5 个日志文件
project.ERROR "log/error.log", 10M * 5 ~ "log/error-%d(%Y%m%d).#2s.log"; log_format
```

## 3. 示例

`main.c` 文件示例：

```C
#include <stdio.h>
#include <stdlib.h>
#include <sys/stat.h>

#include "zlog.h"

int main()
{
    int rc;
    zlog_category_t *zc;


    // 确保 log 目录存在，避免 zlog 写入文件时因目录不存在而失败
    mkdir("log", 0777);

    // 初始化 zlog，加载配置文件
    rc = zlog_init("zlog.conf");
    if (rc) {
        printf("ERROR: Failed to init zlog;\r\n");
        return -1;
    }

    // 获取配置文件中定义的日志分类 "project"
    zc = zlog_get_category("project");
    if (!zc) {
        printf("ERROR: Failed to get zlog category;\r\n");
        zlog_fini();
        return -1;
    }


    // 正常信息，只会打印到屏幕
    zlog_info(zc, "System info");

    // 致命错误，会同时打印到屏幕并追加写入 log/error.log 文件
    zlog_fatal(zc, "Fatel error");

    // 程序结束前，清理 zlog 释放内存
    zlog_fini();

    return 0;
}
```

## 4. 将 Zlog 集成到 CMake

可以在 `CMakeLists.txt` 末尾加入拷贝命令，这样 `zlog.conf` 就会复制到 `build` 目录下，避免程序找不到配置文件。

```CMake
# 查找线程库
set(THREADS_PREFER_PTHREAD_FLAG ON)
find_package(Threads REQUIRED)

# 查找 zlog 头文件和库文件
find_path(ZLOG_INCLUDE_DIR zlog.h 
    PATHS /usr/local/include /usr/include
)

find_library(ZLOG_LIBRARY NAMES zlog 
    PATHS /usr/local/lib /usr/lib
)

if(NOT ZLOG_INCLUDE_DIR OR NOT ZLOG_LIBRARY)
    message(FATAL_ERROR "Could not find zlog library. Please ensure zlog is installed.")
else()
    message(STATUS "Found zlog include dir at: ${ZLOG_INCLUDE_DIR}")
    message(STATUS "Found zlog library at: ${ZLOG_LIBRARY}")
endif()

# 追加头文件
target_include_directories(${PROJECT_NAME} PRIVATE 
    ${ZLOG_INCLUDE_DIR}
)

# 链接三方库
target_link_libraries(${PROJECT_NAME} PRIVATE 
    Threads::Threads 
    ${ZLOG_LIBRARY}
)

# 执行拷贝命令
add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD 
    COMMAND ${CMAKE_COMMAND} -E copy 
    ${CMAKE_CURRENT_SOURCE_DIR}/zlog.conf 
    ${CMAKE_CURRENT_BINARY_DIR}/zlog.conf
)
```

## 5. Zlog 日志级别

Zlog 默认日志级别有6个，如下表：

| 种类          | 等级值 | 宏                   |
| ------------ | --- | ----------------------- |
| FATAL (致命)  | 120 | `zlog_fatal(cat, ...)`  |
| ERROR (错误)  | 100 | `zlog_error(cat, ...)`  |
| WARN (警告)   | 80  | `zlog_warn(cat, ...)`   |
| NOTICE (注意) | 60  | `zlog_notice(cat, ...)` |
| INFO (信息)   | 40  | `zlog_info(cat, ...)`   |
| DEBUG (调试)  | 20  | `zlog_debug(cat, ...)`  |

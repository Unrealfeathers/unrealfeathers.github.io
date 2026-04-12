---
title: libuv 安装
date:  2026-04-07 20:38:37

math: false
mermaid: false
category_bar: false

categories:
  - Linux

tags:
  - libuv
  - CMake

excerpt: "libuv 是 Node.js 底层的 C 语言异步 I/O 库"
permalink: /posts/20260407-203837.html
---
## 1. 安装

> [libuv/libuv: Cross-platform asynchronous I/O](https://github.com/libuv/libuv)

环境如下：

- Ubuntu Server 24.04 LTS

安装步骤如下：

```Bash
sudo apt install libuv1-dev
```

## 2. 将 libuv 集成到 CMake

```CMake
# 查找 libuv 头文件和库文件
find_path(UV_INCLUDE_DIR uv.h 
    PATHS /usr/local/include /usr/include
)

find_library(UV_LIBRARY NAMES uv 
    PATHS /usr/local/lib /usr/lib /usr/lib/x86_64-linux-gnu
)

if(NOT UV_INCLUDE_DIR OR NOT UV_LIBRARY)
    message(FATAL_ERROR "Could NOT find libuv library. Please ensure libuv is installed.")
else()
    message(STATUS "Found libuv include dir at: ${UV_INCLUDE_DIR}")
    message(STATUS "Found libuv library at: ${UV_LIBRARY}")
endif()

# 追加头文件
target_include_directories(${PROJECT_NAME} PRIVATE 
    ${UV_INCLUDE_DIR}
)

# 链接三方库
target_link_libraries(${PROJECT_NAME} PRIVATE 
    ${UV_LIBRARY}
)
```

## 3. 开发注意事项

### 3.1 `uv_default_loop()`

`libuv` 是基于事件驱动的。所有的网络读写、定时器等操作都需要挂载到一个事件循环（Event Loop）上。使用此函数获取系统的默认事件循环实例，并赋值给全局或静态变量；

### 3.2 `uv_udp_bind(..., ..., UV_UDP_REUSEADDR)`

`UV_UDP_REUSEADDR` 这个标志位告诉操作系统开启端口复用。这在开发和生产环境中非常重要：如果程序崩溃或重启，端口通常会处于 `TIME_WAIT` 等状态被系统保留一段时间。开启此选项可以允许程序立即再次绑定该端口，避免**端口已被占用**的错误；

### 3.3 `uv_run(loop, UV_RUN_DEFAULT)`

这是一个阻塞型函数，是启动整个事件循环的马达。一旦调用，它就会在一个死循环中不断检查是否有网络事件、定时器事件发生，直到没有任何活动的句柄为止。通常的做法是在主线程完成所有模块的初始化后，再拉起一个专门的线程去执行 `uv_run()`；

{% note danger %}
当把 `uv_run()` 放到了独立的后台线程中，架构就变成了多线程环境。这个时候必须遵守 `libuv` 最重要的一条铁律：

**`libuv` 的事件循环和它上面的所有句柄都是非线程安全的！**

这意味着绝对不能在主线程（或者其他工作线程）里直接调用 `uv_udp_send()` 向外发数据，或者调用 `uv_close()` 关闭 socket；
{% endnote %}

跨线程通信：

如果需要在主线程触发网络层发送数据，必须使用 `libuv` 提供的唯一线程安全机制：`uv_async_t`。它的工作原理是：

1. 在初始化阶段，向事件循环注册一个 `uv_async_t` 句柄和一个回调函数；

2. 当主线程想要发送数据时，主线程将数据放入一个线程安全的队列（例如：加锁的链表），然后调用 `uv_async_send(&async_handle)`；

3. `uv_async_send()` 会以**线程安全**的方式唤醒后台的事件循环线程；

4. 事件循环线程被唤醒后，会在自己的线程上下文中执行之前注册好的 async 回调函数。在这个回调函数里，再从队列中取出数据，并安全地调用 `uv_udp_send()`；

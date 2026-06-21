---
title: C 程序内存调试方法
date:  2026-05-17 15:27:22

math: false
mermaid: false
category_bar: false

categories:
  - Linux

tags:
  - C
  - Massif
  - memcheck
  - Valgrind

excerpt: "对于一个 7x24 小时运行的 C 程序而言，内存调试是十分重要的，毕竟指针是这样的"
permalink: /posts/20260517-152722.html
---
## 1. `top` 命令

在 Linux 系统中，`top` 命令是实时查看系统资源使用情况的核心工具。首先使用 `top` 查看所有进程的 PID，找到调试程序的 PID。

然后使用以下命令查看指定程序的系统资源使用情况：

```Bash
top -p <PID>
```

{% note info %}
`-p <PID>`：监控指定 PID 的进程；
{% endnote %}

表头含义如下：

- PID (Process ID)：进程号，系统分配给该进程的唯一数字标识符；

- USER：用户名，启动并拥有该进程的系统用户；

- PR (Priority)：进程优先级，操作系统进行任务调度时的优先级；数字越小，优先级越高；

- NI (Nice value)：Nice 值，用来影响非实时进程优先级的数值；范围通常是 -20 到 19；**负值** 表示优先级较高（占用更多 CPU 时间），**正值** 表示优先级较低（对其他进程更“友好/Nice”）；

- VIRT (Virtual Memory Size)：虚拟内存大小，进程申请使用的虚拟内存总量，包括代码、数据、共享库以及已经分配但尚未使用的内存页面；单位通常是 KB；

- RES (Resident Set Size)：常驻内存大小，进程当前实际使用的物理内存大小，不包括被换出到交换区的内存；单位通常是 KB，**排查内存泄漏时重点看此项**；

- SHR (Shared Memory Size)：共享内存大小，该进程与其他进程共享的内存空间大小；单位通常是 KB；

- S (Status)：进程状态，用单个字母表示进程当前的运行情况：
    - R (Running)：正在运行或在队列中等待运行；
    - S (Sleeping)：处于休眠状态（可中断），通常是在等待某个事件完成或等待用户输入；
    - D (Disk Sleep)：不可中断的休眠状态，通常是在等待 I/O 操作完成；如果系统中出现大量 `D` 状态进程，通常意味着磁盘或存储出现了瓶颈；
    - T (Stopped)：进程被暂停或处于跟踪状态；
    - Z (Zombie)：僵尸进程，进程已经运行结束，但是其父进程还没有回收它的退出状态；

- %CPU：CPU 使用率，自上次 `top` 刷新以来，该进程占用 CPU 时间的百分比；如果系统有多个 CPU 核心，该值有可能超过 100%；

- %MEM：内存使用率，该进程的物理内存占用（RES）占系统总物理内存的百分比；

- TIME+：累计 CPU 时间，自该进程启动以来，累计占用的总 CPU 时间；精确到百分之一秒，格式通常为 `分钟:秒.百分之一秒`；

- COMMAND：命令名称，启动该进程的具体命令或程序名称；按下 `c` 键可以在命令名称和完整的命令行路径之间切换；

在排查多线程程序时，可以使用以下命令：

```Bash
top -Hp <PID>
```

{% note info %}
`-H`：开启线程模式；

默认情况下，`top` 命令把一个进程的所有线程资源汇总在一起，只显示一行 “进程” 级别的统计信息。加上 `-H` 参数后，它会将该进程展开，把内部的每一个 “线程” 作为单独的一行显示出来。
{% endnote %}

此时，第一列的 PID 实际上是线程 ID。在 Linux 底层，线程通常被实现为轻量级进程。因此，在这个模式下，第一列显示的数字是线程的 ID，而不是该程序的总进程 ID。`%CPU` 和 `%MEM` 列显示的不再是整个进程的汇总数据，而是单个具体线程的资源消耗率。

## 2. Valgrind

### 2.1 Memcheck 工具

Memcheck 是 Valgrind 中最常用的工具，用于检测各种内存问题。

在 `CMakeLists.txt` 中末尾添加以下命令，编译后使用 `make memcheck` 来进行内存调试：

```CMake
# Valgrind 调试命令
add_custom_target(memcheck 
    COMMAND valgrind 
            --tool=memcheck 
            --leak-check=full 
            --show-leak-kinds=all 
            --track-origins=yes 
            --suppressions=${CMAKE_SOURCE_DIR}/valgrind_ignore.supp 
            --log-file=${CMAKE_BINARY_DIR}/valgrind_report.log 
            $<TARGET_FILE:${PROJECT_NAME}> 
    WORKING_DIRECTORY ${CMAKE_BINARY_DIR} 
    DEPENDS ${PROJECT_NAME} 
    COMMENT "Running Valgrind with module suppressions..."
)
```

{% note info %}
注：这里假设可执行目标名称与项目名称 `${PROJECT_NAME}` 相同。如果不同，请将其替换为实际的 `target` 名称。
{% endnote %}

运行参数解释：

- `--tool=memcheck`：指定调试工具为 `memcheck`；

- `--leak-check=full`：开启详细的内存泄漏检查，并在日志中输出详细的泄漏发生位置；

- `--show-leak-kinds=all`：显示所有类型的内存泄漏情况，包括：definite、indirect、possible 和 reachable；

- `--track-origins=yes`：追踪未初始化内存的具体来源；

- `--suppressions=<ignore_path>`：设置调试忽略规则文件路径；

- `--log-file=<log_path>`：设置调试日志输出路径；

### 2.2 Massif 工具

使用 Massif 工具排查**逻辑泄漏**或**内存膨胀**，是一个非常经典的手段。它不会像 Memcheck 那样去抓哪里忘记 `free`，而是每隔一段时间给程序堆内存拍个快照，最后帮你找出在内存占用最高峰时，到底是哪行代码申请了最多的内存。

#### 2.2.1 进行调试编译

为了让 Massif 能够精准展示哪个文件的哪一行代码分配了内存，在编译时必须带上调试符号，并且关掉高级别的优化。

- 必须加上的编译选项：`-g`

- 建议的优化级别：`-O0` 或 `-O1`

#### 2.2.2 使用 Massif 运行程序

不要直接启动程序，而是把它交给 Valgrind 托管运行。打开终端，输入以下命令：

```Bash
valgrind --tool=massif --time-unit=B --stacks=yes ./program_path
```

运行参数解释：

- `--tool=massif`：指定调试工具为 `massif`；

- `--time-unit=B`：Massif 默认以**执行的指令数**作为时间轴。改成 B(Bytes, 已分配的字节数) 后，生成的图表横坐标会更直观；

- `--stacks=yes`：设置同时监控栈内存的分配；

{% note warning %}
程序在 Massif 下运行会非常慢。你需要耐心等待，并且想办法触发那个导致内存不断上涨的业务逻辑。

等内存涨到一定程度后，按 `Ctrl+C` 正常结束程序，或者让它自然退出。
{% endnote %}

#### 2.2.3 分析日志

程序退出后，在当前目录下会生成一个新文件，名字为 `massif.out.PID`，可以使用两种方式对其进行解析：

1. 使用命令行工具 ms_print

```Bash
ms_print massif.out.PID | less
```

2. 使用图形化界面 massif-visualizer

安装并启动 massif-visualizer，然后导入文件。它会生成一张非常漂亮的彩色折线图和可交互的火焰图/调用树。

#### 2.2.4 追踪进程的总内存

如果程序大量使用了 mmap 或者第三方底层库，默认的 Massif 可能抓不到这些非 malloc/new 分配的内存。这个时候可以使用以下命令：

```Bash
valgrind --tool=massif --pages-as-heap=yes ./program_path
```

运行参数解释：

- `--pages-as-heap=yes`：这个参数会让 Massif 以操作系统底层**页分配**的角度去监控内存，几乎 100% 贴合 `top` 命令里的 RES 涨幅，但输出的调用栈会更底层一些。

## 3. Address Sanitizer 工具

Address Sanitizer 是 GCC 和 Clang 编译器内置的现代内存错误检测工具。相比于 Valgrind Memcheck，ASan 的运行速度极快，非常适合在大型项目或需要长时间跑满负载的服务中开启。

它可以精准捕捉以下常见内存错误：内存越界访问、释放后使用、内存泄漏、多次释放；

### 3.1 编译与配置

要使用 ASan，无需像 Valgrind 那样通过外部工具启动程序，只需在编译和链接阶段加上特定的标志即可。在 CMakeLists.txt 中可以这样配置：

```CMake
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fsanitize=address -fno-omit-frame-pointer -g")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fsanitize=address -fno-omit-frame-pointer -g")
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -fsanitize=address -static-libasan")
```

{% note info %}
`-fno-omit-frame-pointer`：阻止编译器优化掉帧指针，让 ASan 在报错时输出更清晰、更完整的函数调用栈；
{% endnote %}

### 3.2 运行和分析

编译完成后，直接正常运行程序即可。程序运行结束或者手动使用 `Ctrl + C` 结束后，ASan 会在终端输出一份带有颜色高亮的详细崩溃报告。

## 4. perf 工具

如果需要直接追踪缺页中断，可以使用 Linux 的性能分析工具 `perf`。同样的，为了抓取完整的函数调用栈，建议在编译时加入 `-fno-omit-frame-pointer` 参数。

```Bash
# 运行程序后，开启追踪
sudo perf record -e page-faults -g -p <PID>

# 结束程序后，获取报告
sudo perf report
```

这会直接展示出是哪些函数触发了 4KB 物理内存的实际映射。
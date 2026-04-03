---
title: "warning: _close is not implemented and will always fail"
date:  2026-03-28 20:39:22

math: false
mermaid: false
category_bar: false

categories:
  - Embedded System

tags:
  - C
  - CMake

excerpt: "in function `_close_r': warning: _close is not implemented and will always fail"
permalink: /posts/20260328-203922.html
---
## 1. 报错信息

```Text
/usr/lib/gcc/arm-none-eabi/13.2.1/../../../arm-none-eabi/bin/ld: /usr/lib/gcc/arm-none-eabi/13.2.1/../../../arm-none-eabi/lib/thumb/v7-m/nofp/libg_nano.a(libc_a-closer.o): in function `_close_r':
/build/newlib-38V0JC/newlib-4.4.0.20231231/build_nano/arm-none-eabi/thumb/v7-m/nofp/newlib/../../../../../../newlib/libc/reent/closer.c:47:(.text+0xc): warning: _close is not implemented and will always fail
/usr/lib/gcc/arm-none-eabi/13.2.1/../../../arm-none-eabi/bin/ld: /usr/lib/gcc/arm-none-eabi/13.2.1/../../../arm-none-eabi/lib/thumb/v7-m/nofp/libg_nano.a(libc_a-closer.o): note: the message above does not take linker garbage collection into account

/usr/lib/gcc/arm-none-eabi/13.2.1/../../../arm-none-eabi/bin/ld: /usr/lib/gcc/arm-none-eabi/13.2.1/../../../arm-none-eabi/lib/thumb/v7-m/nofp/libg_nano.a(libc_a-lseekr.o): in function `_lseek_r':
/build/newlib-38V0JC/newlib-4.4.0.20231231/build_nano/arm-none-eabi/thumb/v7-m/nofp/newlib/../../../../../../newlib/libc/reent/lseekr.c:49:(.text+0x14): warning: _lseek is not implemented and will always fail
/usr/lib/gcc/arm-none-eabi/13.2.1/../../../arm-none-eabi/bin/ld: /usr/lib/gcc/arm-none-eabi/13.2.1/../../../arm-none-eabi/lib/thumb/v7-m/nofp/libg_nano.a(libc_a-lseekr.o): note: the message above does not take linker garbage collection into account

/usr/lib/gcc/arm-none-eabi/13.2.1/../../../arm-none-eabi/bin/ld: /usr/lib/gcc/arm-none-eabi/13.2.1/../../../arm-none-eabi/lib/thumb/v7-m/nofp/libg_nano.a(libc_a-readr.o): in function `_read_r':
/build/newlib-38V0JC/newlib-4.4.0.20231231/build_nano/arm-none-eabi/thumb/v7-m/nofp/newlib/../../../../../../newlib/libc/reent/readr.c:49:(.text+0x14): warning: _read is not implemented and will always fail
/usr/lib/gcc/arm-none-eabi/13.2.1/../../../arm-none-eabi/bin/ld: /usr/lib/gcc/arm-none-eabi/13.2.1/../../../arm-none-eabi/lib/thumb/v7-m/nofp/libg_nano.a(libc_a-readr.o): note: the message above does not take linker garbage collection into account

/usr/lib/gcc/arm-none-eabi/13.2.1/../../../arm-none-eabi/bin/ld: /usr/lib/gcc/arm-none-eabi/13.2.1/../../../arm-none-eabi/lib/thumb/v7-m/nofp/libg_nano.a(libc_a-writer.o): in function `_write_r':
/build/newlib-38V0JC/newlib-4.4.0.20231231/build_nano/arm-none-eabi/thumb/v7-m/nofp/newlib/../../../../../../newlib/libc/reent/writer.c:49:(.text+0x14): warning: _write is not implemented and will always fail
/usr/lib/gcc/arm-none-eabi/13.2.1/../../../arm-none-eabi/bin/ld: /usr/lib/gcc/arm-none-eabi/13.2.1/../../../arm-none-eabi/lib/thumb/v7-m/nofp/libg_nano.a(libc_a-writer.o): note: the message above does not take linker garbage collection into account
```

警告原因：

编译链接了精简版的 C 标准库（`nano.specs`，即输出中的 `libg_nano.a`）。当使用了某些依赖系统的标准 C 函数（如 `printf`、`malloc`）时，该库会尝试调用底层的系统调用接口（如 `_write`）。但在无操作系统的裸机或者简单的 RTOS 环境中，这些底层接口是不存在的。

即使在编译链接选项中引入 `nosys.specs`，提供了 `_read`、`_write` 等系统调用的**空壳实现**（让程序能顺利链接，不至于报 `undefined reference` 致命错误）。但是，GCC 的开发者故意在这些空壳函数内部打上了警告属性（通过编译器的 `.gnu.warning` 特性）。

它的潜台词是：“我替你填补了这些底层函数，让你能通过编译；但我得警告你，这些函数是个空壳，如果你的代码真的执行到这里，它一定会失败。”

## 2. 解决办法

既然标准库自带的空壳函数太“吵”，最彻底的解决办法就是自己提供一套“安静”的空壳函数。因为 C 语言链接器的规则是：用户自定义的函数优先级高于系统库函数。只要你的工程里有这些函数，链接器就不会再去 `libg_nano.a` 里找，自然也就没有警告了。

代码如下：

{% fold info @Code %}
```C
/* sys_calls.c - 提供静默的底层系统调用实现以消除编译警告 */

#include <unistd.h>
#include <sys/stat.h>

__attribute__((weak)) int _write(int file, char *ptr, int len)
{
    return len;
}

__attribute__((weak)) int _read(int file, char *ptr, int len)
{
    return 0;
}

__attribute__((weak)) int _close(int file)
{
    return -1;
}

__attribute__((weak)) int _lseek(int file, int ptr, int dir)
{
    return 0;
}

__attribute__((weak)) int _fstat(int file, struct stat *st)
{
    st->st_mode = S_IFCHR;
    return 0;
}

__attribute__((weak)) int _isatty(int file)
{
    return 1;
}

__attribute__((weak)) int _getpid(void)
{
    return 1;
}

__attribute__((weak)) int _kill(int pid, int sig)
{
    return -1;
}
```
{% endfold %}

{% note info %}
函数加上了 `__attribute__((weak))` 弱引用标记。这样以后如果你在别的地方真正实现了相应的函数，不需要删掉这段代码，你的新代码会自动覆盖它。
{% endnote %}

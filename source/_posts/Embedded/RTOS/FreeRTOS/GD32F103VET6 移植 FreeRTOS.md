---
title: GD32F103VET6 移植 FreeRTOS
date:  2026-04-03 18:12:36

math: false
mermaid: false
category_bar: false

categories:
  - Embedded System

tags:
  - GD32
  - RTOS
  - FreeRTOS

excerpt: "移植 FreeRTOS 到 Linux 开发、编译环境"
permalink: /posts/20260403-181236.html
---
## 1. 开发环境准备

> [Unrealfeathers/hello_led](https://github.com/Unrealfeathers/hello_led)
> [GD32F103VET6 Linux 开发和调试环境部署 | Unrealfeathers' Blog](https://flowerdown.org/posts/20260329-230857)

GitHub 仓库中有示例代码，Linux 下的开发环境搭建请参考我的另一篇文章。

## 2. 获取 FreeRTOS Kernel

> [FreeRTOS/FreeRTOS-Kernel](https://github.com/FreeRTOS/FreeRTOS-Kernel/tree/main)

在项目根目录下的 `lib` 目录中创建 `FreeRTOS` 目录，用于存放 FreeRTOS Kernel 源代码，目录结构如下：

```Text
hello_led/
├── ......
├── lib/                        # 第三方库 / 厂商固件库
│   ├── CMSIS/                  # ARM Cortex 核心文件及 GD32 启动代码/系统文件
│   ├── FreeRTOS/               # FreeRTOS Kernel 源代码
│   │   ├── include/
│   │   └── source/
│   └── GD32F10x_standard_peripheral/ # GD32 标准外设库
└── ......
```

`include` 目录下，拷贝 FreeRTOS Kernel 仓库 `include` 目录下所有的 `.h` 文件：

{% fold info @目录结构 %}
```Text
include/
├── FreeRTOS.h
├── StackMacros.h
├── atomic.h
├── croutine.h
├── deprecated_definitions.h
├── event_groups.h
├── list.h
├── message_buffer.h
├── mpu_prototypes.h
├── mpu_syscall_numbers.h
├── mpu_wrappers.h
├── newlib-freertos.h
├── picolibc-freertos.h
├── portable.h
├── projdefs.h
├── queue.h
├── semphr.h
├── stack_macros.h
├── stream_buffer.h
├── task.h
└── timers.h
```
{% endfold %}

`source` 目录下，拷贝 FreeRTOS Kernel 仓库根目录下所有的 `.c` 文件，以及 `portable` 目录下的对应文件：

{% fold info @目录结构 %}
```Text
include/
├── portable/
│   ├── GCC/ARM_CM3/
│   │   ├── port.c
│   │   └── portmacro.h
│   └── MemMang/
│       └── heap_4.c
├── croutine.c
├── event_groups.c
├── list.c
├── queue.c
├── stream_buffer.c
├── tasks.c
└── timers.c
```
{% endfold %}

## 3. 准备 FreeRTOSConfig.h

> [FreeRTOS/FreeRTOS](https://github.com/FreeRTOS/FreeRTOS)

可以从 FreeRTOS 源码的 `FreeRTOS/Demo` 目录找到相似型号的示例从中拷贝一份，例如：`CORTEX_STM32F103_GCC_Rowley/`。

我的示例如下：

{% fold info @FreeRTOSConfig.h %}
```C
#ifndef FREERTOS_CONFIG_H
#define FREERTOS_CONFIG_H

#include "gd32f10x.h"

extern uint32_t SystemCoreClock;

/*-----------------------------------------------------------
 * 核心基础配置
 *----------------------------------------------------------*/
#define configUSE_PREEMPTION                    1
#define configUSE_TIME_SLICING                  1
#define configUSE_IDLE_HOOK                     0
#define configUSE_TICK_HOOK                     0
#define configCPU_CLOCK_HZ                      ( SystemCoreClock )
#define configTICK_RATE_HZ                      ( ( TickType_t ) 1000 )
#define configMAX_PRIORITIES                    ( 5 )
#define configMINIMAL_STACK_SIZE                ( ( unsigned short ) 128 )
#define configMAX_TASK_NAME_LEN                 ( 16 )
#define configUSE_16_BIT_TICKS                  0
#define configIDLE_SHOULD_YIELD                 1

/*-----------------------------------------------------------
 * 内存与堆栈分配 (GD32F103VET6 有 64KB SRAM)
 *----------------------------------------------------------*/
#define configSUPPORT_DYNAMIC_ALLOCATION        1
#define configTOTAL_HEAP_SIZE                   ( ( size_t ) ( 48 * 1024 ) )
#define configCHECK_FOR_STACK_OVERFLOW          2

/*-----------------------------------------------------------
 * 调试与断言
 *----------------------------------------------------------*/
#define configUSE_TRACE_FACILITY                1
// 发生致命错误或传参非法时，直接关闭中断并死循环，方便 GDB 抓住现场
#define configASSERT( x ) if( ( x ) == 0 ) { taskDISABLE_INTERRUPTS(); for( ;; ); }

/*-----------------------------------------------------------
 * 互斥量与信号量
 *----------------------------------------------------------*/
#define configUSE_MUTEXES                       1
#define configUSE_RECURSIVE_MUTEXES             1
#define configUSE_COUNTING_SEMAPHORES           1
#define configQUEUE_REGISTRY_SIZE               8

/*-----------------------------------------------------------
 * 协程 (已废弃，直接关闭)
 *----------------------------------------------------------*/
#define configUSE_CO_ROUTINES                   0
#define configMAX_CO_ROUTINE_PRIORITIES         ( 2 )

/*-----------------------------------------------------------
 * 开启的 API 函数
 *----------------------------------------------------------*/
#define INCLUDE_vTaskPrioritySet                1
#define INCLUDE_uxTaskPriorityGet               1
#define INCLUDE_vTaskDelete                     1
#define INCLUDE_vTaskCleanUpResources           0
#define INCLUDE_vTaskSuspend                    1
#define INCLUDE_vTaskDelayUntil                 1
#define INCLUDE_vTaskDelay                      1

/*-----------------------------------------------------------
 * Cortex-M3 中断优先级配置 (重点核心)
 *----------------------------------------------------------*/

// GD32F10x 系列的 NVIC 使用 4 位来表示中断优先级
#define configPRIO_BITS                         4

// 库函数可用的最低优先级
#define configLIBRARY_LOWEST_INTERRUPT_PRIORITY         15

/* FreeRTOS 可接管的最高中断优先级，设置为 5
 * 优先级 0~4 的中断不会被 FreeRTOS 屏蔽，适合极高实时性要求的裸机中断，
 * 但在这些极高优先级中断里，绝对不能调用任何带有 "FromISR" 后缀的 FreeRTOS API！ */
#define configLIBRARY_MAX_SYSCALL_INTERRUPT_PRIORITY    5

// 转换为 Cortex-M3 硬件所需的实际左移值
#define configKERNEL_INTERRUPT_PRIORITY         ( configLIBRARY_LOWEST_INTERRUPT_PRIORITY << (8 - configPRIO_BITS) )
#define configMAX_SYSCALL_INTERRUPT_PRIORITY    ( configLIBRARY_MAX_SYSCALL_INTERRUPT_PRIORITY << (8 - configPRIO_BITS) )

/*-----------------------------------------------------------
 * 映射 FreeRTOS 底层中断
 *----------------------------------------------------------*/
#define vPortSVCHandler     SVC_Handler
#define xPortPendSVHandler  PendSV_Handler
#define xPortSysTickHandler SysTick_Handler

#endif /* FREERTOS_CONFIG_H */
```
{% endfold %}

## 4. 映射系统中断

FreeRTOS 需要接管 Cortex-M3 的三个核心系统中断来实现任务上下文切换和系统时钟：`SVC_Handler`、`PendSV_Handler` 和 `SysTick_Handler`。

为了避免去修改启动汇编文件或标准库文件，最优雅的做法是在 `FreeRTOSConfig.h` 的末尾添加以下宏定义，将裸机的默认中断名强行映射到 FreeRTOS 的处理函数上：

```C
#define vPortSVCHandler     SVC_Handler
#define xPortPendSVHandler  PendSV_Handler
#define xPortSysTickHandler SysTick_Handler
```

{% note warning %}
如果裸机工程（通常在 `gd32f10x_it.c` 文件中）已经定义了这三个函数的空实现，**必须将它们注释掉或删除**，否则链接器会报重复定义的错误。
{% endnote %}

## 5. 编译测试

根据自己的目录结构更新 `CMakeLists.txt`。

在 `main.c` 中编写测试程序：

{% fold info @Code %}
```C
#include "gd32f10x.h"
#include "gd32f10x_rcu.h"
#include "gd32f10x_gpio.h"

#include "FreeRTOS.h"
#include "task.h"

/**
 * @brief 栈溢出钩子函数
 */
void vApplicationStackOverflowHook(TaskHandle_t xTask, char *pcTaskName)
{
    // 发生栈溢出时，进入死循环
    while(1);
}

/**
 * @brief 初始化 PC13 为推挽输出
 */
void gpio_config()
{
    // 开启 GPIOC 的外设时钟
    rcu_periph_clock_enable(RCU_GPIOC);

    /* 配置 PC13 引脚模式:
     * GPIO_MODE_OUT_PP: 推挽输出
     * GPIO_OSPEED_50MHZ: 50MHz 最大翻转频率
     * GPIO_PIN_13: 指定 13 号引脚
     */
    gpio_init(GPIOC, GPIO_MODE_OUT_PP, GPIO_OSPEED_50MHZ, GPIO_PIN_13);
}

/**
 * @brief FreeRTOS LED 闪烁测试任务
 */
void vTaskLedBlink(void *pvParameters)
{
    while (1) {
        // Reset (低电平) 点亮 LED
        gpio_bit_reset(GPIOC, GPIO_PIN_13);
        vTaskDelay(pdMS_TO_TICKS(1000));

        // Set (高电平) 熄灭 LED
        gpio_bit_set(GPIOC, GPIO_PIN_13);
        vTaskDelay(pdMS_TO_TICKS(1000));
    }
}

int main()
{
    // 初始化 GPIO
    gpio_config();

    /* 创建 LED 闪烁任务
     * 参数1: 任务函数指针
     * 参数2: 任务名称
     * 参数3: 任务栈大小（单位：字 = 4 字节）
     * 参数4: 传递给任务的参数
     * 参数5: 任务优先级
     * 参数6: 任务句柄
     */
    xTaskCreate(vTaskLedBlink, 
                "LED_BLINK", 
                configMINIMAL_STACK_SIZE, 
                NULL, 
                tskIDLE_PRIORITY + 1, 
                NULL);

    vTaskStartScheduler();

    // 死循环拦截
    while(1);
}
```
{% endfold %}

按下 `F5` 编译、烧录后，查看 LED 闪烁是否正常。

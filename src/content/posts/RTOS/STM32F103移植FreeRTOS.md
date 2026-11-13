---
title: "STM32F103 移植 FreeRTOS V11.3.0"
published: 2026-07-20
description: "基于 STM32F103ZET6 与 FreeRTOS V11.3.0，从标准外设库工程到多任务点灯的完整移植记录。"
category: "RTOS"
tags: [FreeRTOS, STM32F103, Cortex-M3, Keil]
draft: false
---

# STM32F103 V3.6 + FreeRTOS V11.3.0 移植全流程总结

---

## 第一阶段：准备工作

| 步骤  | 操作                                         | 说明         |
| --- | ------------------------------------------ | ---------- |
| 1   | 下载 **STM32F10x 标准外设库 V3.6**                | ST 官方固件库   |
| 2   | 下载 **FreeRTOS 源码 V11.3.0**                 | 开源 RTOS 内核 |
| 3   | 安装 **Keil MDK v5**                         | 集成开发环境     |
| 4   | 准备 **STM32F103ZET6 开发板** + **ST-Link 下载器** | 硬件平台       |

**STM32F10x 标准外设库 V3.6**
https://www.st.com.cn/zh/embedded-software/stsw-stm32054.html#overview
**FreeRTOS内核源码下载 V11.3.0**
https://github.com/FreeRTOS/FreeRTOS-Kernel/releases/tag/V11.3.0


---

## 第二阶段：创建项目目录结构


```text
STM32F103ZET6_FreeRTOS/
├── CORE/                      # 核心文件（启动文件 + core_cm3）
│   ├── startup_stm32f10x_hd.s # 启动文件
│   ├── core_cm3.c
│   └── core_cm3.h
├── Drivers/
│   ├── CMSIS/                 # Cortex-M3 内核支持
│   │   └── CM3/
│   │       ├── CoreSupport/
│   │       └── DeviceSupport/ST/STM32F10x/
│   ├── Startup/               # 启动文件（备用）
│   └── STM32F10x_StdPeriph_Driver/   # 标准外设库
│       ├── inc/               # 头文件
│       └── src/               # 源文件
├── FreeRTOS/
│   ├── Source/                # 内核源码
│   │   ├── tasks.c
│   │   ├── queue.c
│   │   ├── list.c
│   │   ├── timers.c
│   │   ├── event_groups.c
│   │   ├── stream_buffer.c
│   │   ├── croutine.c
│   │   └── include/           # 所有 .h 头文件
│   │       ├── FreeRTOS.h
│   │       ├── task.h
│   │       ├── queue.h
│   │       ├── semphr.h
│   │       └── ...
│   └── Portable/
│       ├── RVDS/ARM_CM3/
│       │   ├── port.c
│       │   └── portmacro.h
│       └── MemMang/
│           └── heap_4.c       # 内存管理
├── User/                      # 用户代码
│   ├── main.c
│   ├── FreeRTOSConfig.h
│   ├── stm32f10x_conf.h
│   ├── stm32f10x_it.c
│   └── stm32f10x_it.h
│
├── Objects/                   # 编译输出
├── Listings/                  # 列表文件
└── Project Files/             # .uvprojx, .uvoptx
```
---

## 第三阶段：复制文件到项目目录

### 3.1 新建空文件夹，从标准库复制

|     | 源路径（标准库）                                                                            |
| :-- | :---------------------------------------------------------------------------------- |
| 1   | `Libraries/CMSIS/CM3/CoreSupport/`                                                  |
| 2   | `Libraries/CMSIS/CM3/DeviceSupport/ST/STM32F10x/`                                   |
| 3   | `Libraries/STM32F10x_StdPeriph_Driver/`                                             |
| 4   | `Libraries\CMSIS\CM3\DeviceSupport\ST\STM32F10x\startup\arm\startup_stm32f10x_hd.s` |
| 5   | `Project\STM32F10x_StdPeriph_Template\main.c`                                       |
| 6   | `Project\STM32F10x_StdPeriph_Template\stm32f10x_it.c`                               |
| 7   | `Project\STM32F10x_StdPeriph_Template\stm32f10x_it.h`                               |
| 8   | `Project\STM32F10x_StdPeriph_Template\stm32f10x_conf.h`                             |

|     | 目标路径（项目）                                        |
| --- | ----------------------------------------------- |
| 1   | `Drivers/CMSIS/CM3/CoreSupport/`                |
| 2   | `Drivers/CMSIS/CM3/DeviceSupport/ST/STM32F10x/` |
| 3   | `Drivers/STM32F10x_StdPeriph_Driver/`           |
| 4   | `CORE/startup_stm32f10x_hd.s`                   |
| 5   | `User/main.c`                                   |
| 6   | `User/stm32f10x_it.c`                           |
| 7   | `User/stm32f10x_it.h`                           |
| 8   | `User/stm32f10x_conf.h`                         |

### 3.2 从 FreeRTOS 源码复制

| 源路径（FreeRTOS）                        | 目标路径（项目）                             |
| ------------------------------------ | ------------------------------------ |
| `FreeRTOS/*.c`（tasks.c, queue.c 等）   | `FreeRTOS/Source/`                   |
| `FreeRTOS/include/`（所有 .h）           | `FreeRTOS/Source/include/`           |
| `FreeRTOS/Portable/RVDS/ARM_CM3/`    | `FreeRTOS/Portable/RVDS/ARM_CM3/`    |
| `FreeRTOS/Portable/MemMang/heap_4.c` | `FreeRTOS/Portable/MemMang/heap_4.c` |
关于内存分配的话建议heap_4.c
https://www.freertos.org/Documentation/02-Kernel/02-Kernel-features/09-Memory-management/01-Memory-management

---

## 第四阶段：修改关键文件

### 4.1 修改 `stm32f10x_it.c`

注释掉 `SVC_Handler`、`PendSV_Handler` 和 `SysTick_Handler`（port.c 会接管）：

```c
// void PendSV_Handler(void) { }
// void SysTick_Handler(void) { }
```
 或者直接用以下内容覆盖
```c
/**
  ******************************************************************************
  * @file    Project/STM32F10x_StdPeriph_Template/stm32f10x_it.c 
  * @author  MCD Application Team
  * @version V3.6.0
  * @date    20-September-2021
  * @brief   主中断服务程序。
  *          此文件为所有异常处理程序和
  *          外设中断服务程序提供模板。
  ******************************************************************************
  * @attention
  *
  * Copyright (c) 2011 STMicroelectronics.
  * All rights reserved.
  *
  * This software is licensed under terms that can be found in the LICENSE file
  * in the root directory of this software component.
  * If no LICENSE file comes with this software, it is provided AS-IS.
  *
  ******************************************************************************
  */

/* 包含头文件 --------------------------------------------------------------*/
#include "stm32f10x_it.h"

/** @addtogroup STM32F10x_StdPeriph_Template
  * @{
  */

/* 私有类型定义 -----------------------------------------------------------*/
/* 私有宏定义 ------------------------------------------------------------*/
/* 私有宏函数 -------------------------------------------------------------*/
/* 私有变量 ---------------------------------------------------------*/
/* 私有函数原型 -----------------------------------------------*/
/* 私有函数 ---------------------------------------------------------*/

/******************************************************************************/
/*            Cortex-M3 处理器异常处理程序                                    */
/******************************************************************************/

/**
  * @brief  非屏蔽中断（NMI）异常处理函数。
  * @param  无
  * @retval 无
  */
void NMI_Handler(void)
{
    /* 用户可以在此添加自己的代码 */
}

/**
  * @brief  硬件错误（Hard Fault）异常处理函数。
  * @param  无
  * @retval 无
  */
void HardFault_Handler(void)
{
    /* 当发生硬件错误异常时，进入无限循环 */
    while (1)
    {
    }
}

/**
  * @brief  内存管理（Memory Manage）异常处理函数。
  * @param  无
  * @retval 无
  */
void MemManage_Handler(void)
{
    /* 当发生内存管理异常时，进入无限循环 */
    while (1)
    {
    }
}

/**
  * @brief  总线错误（Bus Fault）异常处理函数。
  * @param  无
  * @retval 无
  */
void BusFault_Handler(void)
{
    /* 当发生总线错误异常时，进入无限循环 */
    while (1)
    {
    }
}

/**
  * @brief  用法错误（Usage Fault）异常处理函数。
  * @param  无
  * @retval 无
  */
void UsageFault_Handler(void)
{
    /* 当发生用法错误异常时，进入无限循环 */
    while (1)
    {
    }
}

/**
  * @brief  系统服务调用（SVCall）异常处理函数。
  * @param  无
  * @retval 无
  */
// void SVC_Handler(void)
// {
//     /* 由 FreeRTOSConfig.h 的 vPortSVCHandler 别名宏接管，保持注释即可 */
// }

/**
  * @brief  调试监控器（Debug Monitor）异常处理函数。
  * @param  无
  * @retval 无
  */
void DebugMon_Handler(void)
{
}

/**
  * @brief  可挂起的系统服务调用（PendSVC）异常处理函数。
  * @param  无
  * @retval 无
  */

// void PendSV_Handler(void)
// {
//     /* 如果使用 FreeRTOS，需要取消注释并调用 vPortPendSVHandler() */
//     /* vPortPendSVHandler(); */
// }


/**
  * @brief  系统节拍定时器（SysTick）中断处理函数。
  * @param  无
  * @retval 无
  */

// void SysTick_Handler(void)
// {
//     /* 如果使用 FreeRTOS，需要取消注释并调用 xPortSysTickHandler() */
//     /* xPortSysTickHandler(); */
// }


/******************************************************************************/
/*                 STM32F10x 外设中断处理程序                                */
/*  在此处添加所使用的（PPP）外设的中断处理程序，                             */
/*  可用外设中断处理函数的名称请参考启动文件                                  */
/*  (startup_stm32f10x_xx.s)。                                               */
/******************************************************************************/

/**
  * @brief  外设 PPP 中断请求处理函数。
  * @param  无
  * @retval 无
  */
/*void PPP_IRQHandler(void)
{
}*/

/**
  * @}
  */ 
```
### 4.2 创建 `User/FreeRTOSConfig.h`


```c
#ifndef FREERTOS_CONFIG_H
#define FREERTOS_CONFIG_H
#include <stdint.h>
#define configUSE_PREEMPTION                    1
#define configCPU_CLOCK_HZ                      72000000UL
#define configTICK_RATE_HZ                      1000
#define configMAX_PRIORITIES                    5
#define configMINIMAL_STACK_SIZE                128
#define configTOTAL_HEAP_SIZE                   8192
#define configUSE_IDLE_HOOK                     0
#define configUSE_TICK_HOOK                     0
#define configUSE_MUTEXES                       1
#define configUSE_COUNTING_SEMAPHORES           1
#define configUSE_TASK_NOTIFICATIONS            1
#define configUSE_16_BIT_TICKS                  0
#define configPRIO_BITS                         4
#define configLIBRARY_LOWEST_INTERRUPT_PRIORITY 15
#define configLIBRARY_MAX_SYSCALL_INTERRUPT_PRIORITY  5
#define configKERNEL_INTERRUPT_PRIORITY         ( 15 << (8 - 4) )
#define configMAX_SYSCALL_INTERRUPT_PRIORITY    ( 5 << (8 - 4) )
/* API 使能（关键！） */
#define INCLUDE_vTaskDelay                      1
#define INCLUDE_vTaskSuspend                    1
#define INCLUDE_vTaskPrioritySet                1
#define INCLUDE_uxTaskPriorityGet               1
#define INCLUDE_vTaskDelete                     1
#define INCLUDE_vTaskDelayUntil                 1
#define INCLUDE_uxTaskGetStackHighWaterMark     1
#define INCLUDE_eTaskGetState                   1
#define INCLUDE_xTaskGetCurrentTaskHandle       1
#define configASSERT( x ) if( ( x ) == 0 ) { for( ;; ); }

/* 中断入口对接（必须有）：把 port.c 里的实现映射到启动文件的中断向量名 */
#define vPortSVCHandler                         SVC_Handler
#define xPortPendSVHandler                      PendSV_Handler
#define xPortSysTickHandler                     SysTick_Handler
#endif
```

---

## 第五阶段：Keil MDK 工程配置

### 5.1 开始配置

启动Keil，预先安装好STM32F103相关软件包
![FreeRTOS_01](/img/posts/2026/07/20/FreeRTOS/FreeRTOS_01.png)

![FreeRTOS_06](/img/posts/2026/07/20/FreeRTOS/FreeRTOS_06.png)

![FreeRTOS_08](/img/posts/2026/07/20/FreeRTOS/FreeRTOS_08.png)

这里去到刚才构建的项目根目录，或者USER文件夹内也行
![FreeRTOS_09](/img/posts/2026/07/20/FreeRTOS/FreeRTOS_09.png)

![FreeRTOS_10](/img/posts/2026/07/20/FreeRTOS/FreeRTOS_10.png)

![FreeRTOS_11](/img/posts/2026/07/20/FreeRTOS/FreeRTOS_11.png)

![FreeRTOS_12](/img/posts/2026/07/20/FreeRTOS/FreeRTOS_12.png)

![FreeRTOS_14](/img/posts/2026/07/20/FreeRTOS/FreeRTOS_14.png)

![FreeRTOS_15](/img/posts/2026/07/20/FreeRTOS/FreeRTOS_15.png)

![FreeRTOS_16](/img/posts/2026/07/20/FreeRTOS/FreeRTOS_16.png)



### 5.2 创建 Groups 并添加文件

| Group                 | 添加的文件                                                                                                   |
| --------------------- | ------------------------------------------------------------------------------------------------------- |
| **CORE**              | `CORE/startup_stm32f10x_hd.s`、`CORE/core_cm3.c`                                                         |
| **Drivers/CMSIS**     | `Drivers\CMSIS\CM3\DeviceSupport\ST\STM32F10x\system_stm32f10x.c`                                       |
| **Drivers/StdPeriph** | `Drivers/STM32F10x_StdPeriph_Driver/src/*.c`                                                            |
| **FreeRTOS/Source**   | `FreeRTOS/Source/tasks.c`、`queue.c`、`list.c`、`timers.c`、`event_groups.c`、`stream_buffer.c`、`croutine.c` |
| **FreeRTOS/Portable** | `FreeRTOS/Portable/RVDS/ARM_CM3/port.c`、`FreeRTOS/Portable/MemMang/heap_4.c`                            |
| **User**              | `User/main.c`、`User/stm32f10x_it.c`                                                                     |

### 5.3 Include Paths（C/C++ 选项卡）

![FreeRTOS_17](/img/posts/2026/07/20/FreeRTOS/FreeRTOS_17.png)

![FreeRTOS_18](/img/posts/2026/07/20/FreeRTOS/FreeRTOS_18.png)

![FreeRTOS_19](/img/posts/2026/07/20/FreeRTOS/FreeRTOS_19.png)


```text
.\CORE
.\Drivers\CMSIS\CM3\CoreSupport
.\Drivers\CMSIS\CM3\DeviceSupport\ST\STM32F10x
.\Drivers\STM32F10x_StdPeriph_Driver\inc
.\FreeRTOS\Source\include
.\FreeRTOS\Portable\RVDS\ARM_CM3
.\User
```
### 5.4 宏定义（Define）

```text
STM32F10X_HD,USE_STDPERIPH_DRIVER
```

![FreeRTOS_21](/img/posts/2026/07/20/FreeRTOS/FreeRTOS_21.png)

此处主要按芯片选择定义。

本工程用 **ZET6（512KB Flash，大容量）**，所以启动文件用 `startup_stm32f10x_hd.s`、宏用 `STM32F10X_HD`。
如果换成 **C8T6（64KB Flash，中容量）**，只需把启动文件换成 `startup_stm32f10x_md.s`、宏改成 `STM32F10X_MD`，其余文件不用动。

### 5.5 取消勾选 Use MicroLIB（Target 选项卡）

这里取消是为了于内存管理配置冲突
![FreeRTOS_23](/img/posts/2026/07/20/FreeRTOS/FreeRTOS_23.png)

### 5.6 修改main.c文件，直接覆盖原本内容

![FreeRTOS_22](/img/posts/2026/07/20/FreeRTOS/FreeRTOS_22.png)

```c
/* Includes ------------------------------------------------------------------*/

#include "stm32f10x.h"
#include "FreeRTOS.h"
#include "task.h"

void vTaskLED(void *pvParameters);
int main(void)
{
    NVIC_PriorityGroupConfig(NVIC_PriorityGroup_4);
    
    // 硬件初始化（GPIO、USART 等）
    // ...
    
    xTaskCreate(vTaskLED, "LED", 128, NULL, 1, NULL);
    vTaskStartScheduler();
    
    while(1);
}
void vTaskLED(void *pvParameters)
{
    (void)pvParameters;
    while(1)
    {
        // LED 亮
        vTaskDelay(pdMS_TO_TICKS(500));
        // LED 灭
        vTaskDelay(pdMS_TO_TICKS(500));
    }
}
```

---

## 第六阶段：编译与下载

| 步骤  | 操作                          |
| --- | --------------------------- |
| 1   | 点击 **Rebuild**（重新编译）        |
| 2   | 确认 0 Error(s), 0 Warning(s) |
| 3   | 配置 ST-Link 调试器              |
| 4   | 点击 **Load (F8)** 下载到开发板     |
| 5   | 观察 LED 是否按任务逻辑闪烁            |

---

## 📊 快速参考表

| 关键操作     | 文件                                   | 要点                                      |
| -------- | ------------------------------------ | --------------------------------------- |
| 启动文件     | `startup_stm32f10x_hd.s`             | 大容量（512KB Flash）                        |
| 中断注释     | `stm32f10x_it.c`                     | 注释 `SVC/PendSV/SysTick` 三个 Handler |
| 配置头文件    | `FreeRTOSConfig.h`                   | `INCLUDE_vTaskDelay = 1`                |
| 内存管理     | `heap_4.c`                           | 推荐，防碎片化                                 |
| 主程序      | `main.c`                             | 硬件初始化 → 创建任务 → 启动调度器                    |
| 宏定义      | `STM32F10X_HD, USE_STDPERIPH_DRIVER` | Keil 中添加                                |
| MicroLIB | 取消勾选                                 | 避免与 heap 冲突                             |

---

## ✅ 验证成功的标志

1. **编译**：0 Errors, 0 Warnings
    
2. **下载**：成功烧录到开发板
    
3. **运行**：LED 按预期闪烁（或串口输出调试信息）
    

---

---
title: "构建 STM32F103 标准库工程"
published: 2026-09-10
description: "从零搭建 STM32F103ZET6 标准外设库工程：目录结构、Keil 配置、宏与启动文件。"
category: "STM32"
tags: [STM32F103, Keil, 标准外设库, 工程模板]
draft: false
---

# 构建 STM32F103 标准库工程
### 第一阶段：准备工作

| 步骤  | 操作                                         | 说明       |
| --- | ------------------------------------------ | -------- |
| 1   | 下载 **STM32F10x 标准外设库 V3.6**                | ST 官方固件库 |
| 2   | 安装 **Keil MDK v5**                         | 集成开发环境   |
| 3   | 准备 **STM32F103ZET6 开发板** + **ST-Link 下载器** | 硬件平台     |

**STM32F10x 标准外设库 V3.6**
https://www.st.com.cn/zh/embedded-software/stsw-stm32054.html#overview

---

### 第二阶段：创建项目目录结构


```text
STM32F103ZET6_Test/
├── CORE/                      # 核心文件（启动文件 + core_cm3）
│   ├── startup_stm32f10x_hd.s # 启动文件
│   ├── core_cm3.c
│   └── core_cm3.h
├── Drivers/
│   ├── CMSIS/                 # Cortex-M3 内核支持
│   │   └── CM3/
│   │       ├── CoreSupport/
│   │       └── DeviceSupport/ST/STM32F10x/
│   └── STM32F10x_StdPeriph_Driver/   # 标准外设库
│       ├── inc/               # 头文件
│       └── src/               # 源文件
│
├── User/                      # 用户代码
│   ├── main.c
│   ├── stm32f10x_conf.h
│   ├── stm32f10x_it.c
│   └── stm32f10x_it.h
│
├── Objects/                   # 编译输出
├── Listings/                  # 列表文件
└── Project Files/             # .uvprojx, .uvoptx
```
---

### 第三阶段：复制文件到项目目录

#### 3.1 新建空文件夹，从标准库复制

解压得文件
stsw-stm32054_v3-6-0\STM32F10x_StdPeriph_Lib_V3.6.0

|     | 源路径（标准库）                                                                            |
| :-- | :---------------------------------------------------------------------------------- |
| 1   | `Libraries/CMSIS/CM3/CoreSupport/`                                                  |
| 2   | `Libraries/CMSIS/CM3/DeviceSupport/ST/STM32F10x/`                                   |
| 3   | `Libraries/STM32F10x_StdPeriph_Driver/`                                             |
| 4   | `Libraries\CMSIS\CM3\CoreSupport\core_cm3.c`                                        |
| 5   | `Libraries\CMSIS\CM3\CoreSupport\core_cm3.h`                                        |
| 6   | `Libraries\CMSIS\CM3\DeviceSupport\ST\STM32F10x\startup\arm\startup_stm32f10x_hd.s` |
| 7   | `Project\STM32F10x_StdPeriph_Template\main.c`                                       |
| 8   | `Project\STM32F10x_StdPeriph_Template\stm32f10x_it.c`                               |
| 9   | `Project\STM32F10x_StdPeriph_Template\stm32f10x_it.h`                               |
| 10  | `Project\STM32F10x_StdPeriph_Template\stm32f10x_conf.h`                             |

|     | 目标路径（项目）                                        |
| --- | ----------------------------------------------- |
| 1   | `Drivers/CMSIS/CM3/CoreSupport/`                |
| 2   | `Drivers/CMSIS/CM3/DeviceSupport/ST/STM32F10x/` |
| 3   | `Drivers/STM32F10x_StdPeriph_Driver/`           |
| 4   | `CORE/`                                         |
| 5   | `CORE/`                                         |
| 6   | `CORE/startup_stm32f10x_hd.s`                   |
| 7   | `User/main.c`                                   |
| 8   | `User/stm32f10x_it.c`                           |
| 9   | `User/stm32f10x_it.h`                           |
| 10  | `User/stm32f10x_conf.h`                         |


---


### 第四阶段：Keil MDK 工程配置

### 4.1 开始配置

启动Keil，预先安装好STM32F103相关软件包
注意这里的Keil版本要确定一下，用老版本不一定有软件包
最简单的就是这个启动界面不一样了

![01](/img/posts/2026/09/11/构建STM32F103工程/01.png)

#### 4.2 新建工程

![02](/img/posts/2026/09/11/构建STM32F103工程/02.png)

![03](/img/posts/2026/09/11/构建STM32F103工程/03.png)

![04](/img/posts/2026/09/11/构建STM32F103工程/04.png)

![05](/img/posts/2026/09/11/构建STM32F103工程/05.png)
#### 4.3 创建 Groups 并添加文件

![06](/img/posts/2026/09/11/构建STM32F103工程/06.png)

![07](/img/posts/2026/09/11/构建STM32F103工程/07.png)



| Group                 | 添加的文件                                                             |
| --------------------- | ----------------------------------------------------------------- |
| **CORE**              | `CORE/startup_stm32f10x_hd.s`、`CORE/core_cm3.c`                   |
| **Drivers/CMSIS**     | `Drivers\CMSIS\CM3\DeviceSupport\ST\STM32F10x\system_stm32f10x.c` |
| **Drivers/StdPeriph** | `Drivers/STM32F10x_StdPeriph_Driver/src/*.c`                      |
| **User**              | `User/main.c`、`User/stm32f10x_it.c`                               |
![08](/img/posts/2026/09/11/构建STM32F103工程/08.png)

![09](/img/posts/2026/09/11/构建STM32F103工程/09.png)

#### 4.4 Include Paths（C/C++ 选项卡）

![10](/img/posts/2026/09/11/构建STM32F103工程/10.png)

```text
.\CORE
.\Drivers\CMSIS\CM3\CoreSupport
.\Drivers\CMSIS\CM3\DeviceSupport\ST\STM32F10x
.\Drivers\STM32F10x_StdPeriph_Driver\inc
.\User
```

![11](/img/posts/2026/09/11/构建STM32F103工程/11.png)

#### 4.5 宏定义（Define）

```text
STM32F10X_HD,USE_STDPERIPH_DRIVER
```

此处按芯片容量选择密度宏：ZET6 为 512KB Flash（大容量），用 `STM32F10X_HD`；C8T6 才是 64KB（中容量），用 `STM32F10X_MD`

#### 4.6 去除测试部分

![12](/img/posts/2026/09/11/构建STM32F103工程/12.png)

---

### 第五阶段：编译与下载

| 步骤  | 操作                          |
| --- | --------------------------- |
| 1   | 点击 **Rebuild**（重新编译）        |
| 2   | 确认 0 Error(s), 0 Warning(s) |
| 3   | 配置 ST-Link 调试器              |
| 4   | 点击 **Load (F8)** 下载到开发板     |

---

### 📊 快速参考表

| 关键操作 | 文件                                   | 要点                                      |
| ---- | ------------------------------------ | --------------------------------------- |
| 启动文件 | `startup_stm32f10x_hd.s`             | 大容量（512KB Flash）                        |
| 中断注释 | `stm32f10x_it.c`                     | 保留模板中的空函数即可 |
| 主程序  | `main.c`                             | 硬件初始化 → `while(1)` 主循环                    |
| 宏定义  | `STM32F10X_HD, USE_STDPERIPH_DRIVER` | Keil 中添加                                |

---

### ✅ 验证成功的标志

1. **编译**：0 Errors, 0 Warnings
    
2. **下载**：成功烧录到开发板
    
3. **运行**：LED 按预期闪烁（或串口输出调试信息）
    

---

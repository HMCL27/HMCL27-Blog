---
title: "IWDG 独立看门狗"
published: 2026-08-08
description: "STM32F103 独立看门狗的分频、重装载与喂狗，超时后直接复位系统。"
category: "STM32"
tags: [IWDG, 看门狗, LSI, 可靠性]
draft: false
---

# IWDG 独立看门狗

```c
#include "iwdg.h"

void IWDG_Init(u8 prer, u16 rlr)
{
    // 使能写访问
    IWDG_WriteAccessCmd(IWDG_WriteAccess_Enable);
    
    // 等待 PVU 标志位清零（表示可以设置预分频器）
    while(IWDG_GetFlagStatus(IWDG_FLAG_PVU) == SET);
    IWDG_SetPrescaler(prer);
    
    // 等待 RVU 标志位清零（表示可以设置重装载值）
    while(IWDG_GetFlagStatus(IWDG_FLAG_RVU) == SET);
    IWDG_SetReload(rlr);
    
    // 使能 IWDG
    IWDG_Enable();
    
    // 立即喂一次狗（可选）
    IWDG_ReloadCounter();
}

```

```c
#ifndef __IWDG_H
#define __IWDG_H	 
#include "sys.h" //在sys.h中已经包含了stm32f10x.h头文件了，所以这里就不需要再包含一次了

void IWDG_Init(u8 prer,u16 rlr);	//初始化IWDG


#endif //__IWDG_H

```

超时时间 = (4 × 2^prer) × rlr / 40 (ms)
（LSI 时钟 = 40kHz）
它独立于主时钟，由芯片内部的 LSI（约 40kHz）驱动；超时前没喂狗就对整个系统产生复位（不是中断，也不能被屏蔽）。
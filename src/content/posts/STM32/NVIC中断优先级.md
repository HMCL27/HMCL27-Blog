---
title: "NVIC 中断优先级与抢占"
published: 2026-08-11
description: "抢占优先级与响应优先级的区别，用时间轴说明中断嵌套与同优先级排队。"
category: "STM32"
tags: [NVIC, 中断, 优先级, Cortex-M3]
draft: false
---

# NVIC 中断优先级与抢占

关于抢占优先级

```c
时间轴 →
─────────────────────────────────────────────────────
主程序 ────────┐
              │ 中断B触发(抢占=1,响应=0)
              ├────────────────┐
              │ 正在执行中断B    │
              │                │ 中断A触发(抢占=0,响应=0)
              │                ├─────────────┐
              │                │ 中断A执行    │ ← 打断中断B
              │                ├─────────────┘
              ├────────────────┘
              │ 继续执行中断B
──────────────┘
```

```c
时间轴 →
─────────────────────────────────────────────────────
主程序 ────┐
          │ 中断A和中断B同时触发(抢占=1,响应A=0,响应B=1)
          ├─────────────┐
          │ 中断A先执行   │ ← 响应优先级高的先执行
          ├─────────────┘
          ├─────────────┐
          │ 中断B后执行   │ ← 不能打断，只能排队
          ├─────────────┘
──────────┘
```

```c
|分组|抢占优先级位数|响应优先级位数|取值范围|
|---|---|---|---|
|NVIC_PriorityGroup_0|0|4|抢占:无, 响应:0-15|
|NVIC_PriorityGroup_1|1|3|抢占:0-1, 响应:0-7|
|NVIC_PriorityGroup_2|2|2|抢占:0-3, 响应:0-3|
|NVIC_PriorityGroup_3|3|1|抢占:0-7, 响应:0-1|
|NVIC_PriorityGroup_4|4|0|抢占:0-15, 响应:无|
```

```c
## 快速记忆

|问题|答案|
|---|---|
|抢占优先级决定什么？        |能否**打断**别人|
|响应优先级决定什么？        |同时触发时**谁先走**|
|数值越小优先级越高？        |**是**|
|相同抢占优先级能互相打断吗？ |**不能**|
|不同抢占优先级能互相打断吗？ |**能**（高打断低）|
```

**配置建议**
```c
// 重要中断（如系统异常保护）
NVIC_InitStructure.NVIC_IRQChannelPreemptionPriority = 0;  // 最高抢占
NVIC_InitStructure.NVIC_IRQChannelSubPriority = 0;

// 普通中断（如按键、串口）
NVIC_InitStructure.NVIC_IRQChannelPreemptionPriority = 2;  // 中等抢占
NVIC_InitStructure.NVIC_IRQChannelSubPriority = 1;

// 低优先级中断（如LED闪烁）
NVIC_InitStructure.NVIC_IRQChannelPreemptionPriority = 3;  // 最低抢占
NVIC_InitStructure.NVIC_IRQChannelSubPriority = 2;
```
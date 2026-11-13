---
title: "Cortex-M3 DWT 周期计数器"
published: 2026-08-06
description: "用 DWT->CYCCNT 做纳秒级计时与函数耗时统计，替代粗糙的延时估算。"
category: "STM32"
tags: [DWT, Cortex-M3, 性能调试, STM32F103]
draft: false
---

# Cortex-M3 DWT 周期计数器

DWT 是 Cortex-M 内的外设之一，用于系统调试和跟踪，它有一个32位的寄存器 CYCCNT，其为向上的计数器，记录内核时钟运行古树，内核时钟跳动一次，该计数器就加1，精度极高，决定内核频率.
内核时钟为多少一般精度就是1/内核时钟频率，F103系列内核时钟一般为72M，精度为1/72M = 14ns。
当CUCCNT溢出后，会清零重新向上计数
要实现延时功能，总共涉及三个寄存器
DEMCR-----------开启DWT功能
DWT——CTRL---开启CYCCNT
DWT_CYCCNT---获取昔日时钟计数值


![内核调试寄存器DEMCR定义](/img/posts/2026/11/08/DWT基础应用/内核调试寄存器DEMCR定义.png)

![DWT——CUCCNT寄存器使用](/img/posts/2026/11/08/DWT基础应用/DWT——CUCCNT寄存器使用.png)

在使能DWT_CYCCNT寄存器之前，先清0。CYCCNT 寄存器地址是 0xE0001004（DWT 基地址 0xE0001000 + 偏移 0x04），复位默认为0，可读写。
即往0xE0001004这个地址写0，就将DWT_CYCCNT清0了。
此计时器最多计60秒，需要在其重置前提取数据，或只在60秒内做阻塞。常用处理微秒与毫秒级的阻塞。


![CYCCNTENA-（DWT_CTRL的第一位)](/img/posts/2026/11/08/DWT基础应用/CYCCNTENA-（DWT_CTRL的第一位%29.png)

CYCCNTENA
它是 DWT_CTRL 寄存器（地址 0xE0001000）的 bit0，写1使能，启用CYCCNT计数器，否则失能

 新建USER下的dwt文件夹，存放
 bsp_dwt.c
 bsp_dwt.h
 
```c
//DEMCR 地址固定为 0xE000EDFC，bit24 是 TRCENA（跟踪总开关）
  #define DEMCR (*(volatile uint32_t*)(0xE000EDFC))
```

```C
#define DEMCR			 (*(volatile uint32_t*)(0xE000EDFC))
#define DWT_CTRL		 (*(volatile uint32_t*)(0xE0001000))
#define DWT_CYCCNT 	 (*(volatile uint32_t*)(0xE0001004))
```

 ```c
 /**
  ******************************************************************************
  * @file       bsp_dwt.c
  * @author     embedfire
  * @version     V1.0
  * @date        2024
  * @brief      DWT定时器函数接口
  ******************************************************************************
  * @attention
  *
  * 实验平台  ：野火 STM32F103C8T6-STM32开发板 
  * 论坛      ：http://www.firebbs.cn
  * 官网      ：https://embedfire.com/
  * 淘宝      ：https://yehuosm.tmall.com/
  *
  ******************************************************************************
  */
  
  #include "dwt/bsp_dwt.h"
  #include "stm32f10x.h"
  
 /**
  * @brief  初始化 DWT计数器
  * @param  None
  * @retval 使用DWT延时函数前，必须调用本函数
  */
  
void DWT_Init(void)
{
	DEMCR |= (uint32_t)DEMCR_TRCENA;
	DWT_CYCCNT = (uint32_t)0U;
	DWT_CTRL |= (uint32_t)DWT_CTRL_CYCCNTENA;
}

 /**
  * @brief  读取当前时间戳
  * @param  None
  * @retval 当前时间戳
  */

  
uint32_t DWT_GetTick(void)
{
	return ((uint32_t)DWT_CYCCNT);

}

 /**
  * @brief  节拍数转化时间间隔(微秒单位）
  * @param  tick:需要转换的节拍数
  * @param  frequency：内核时钟频率
  * @retval 当前时间戳，即DWT_CYCCNT寄存器的值
  */

uint32_t DWT_TickToMicrosecond(uint32_t tick,uint32_t frequency)
{
	return (uint32_t)((1000000.0 / frequency) * tick);
	//这里必须有个小数位，以达到更精确的目的
}


 /**
  * @brief  DWT 计数实现精确延时，32位计数器
  * @param  time:延迟长度，单位1 us
  * @note   无
  * @retval 无
  */

void DWT_DelayUs(__IO uint32_t time)
{
	
	/*将微秒转化为对应的时钟计数值*/
	uint32_t tick_duration = time * (SystemCoreClock/1000000);
	/*刚进入时的计数器值*/
	uint32_t tick_start = DWT_GetTick();
	
	while( DWT_GetTick() - tick_start < tick_duration );
}

 /**
  * @brief  DWT 计数实现精确延时，32位计数器
  * @param  time:延迟长度，单位1 ms
  * @note   无
  * @retval 无
  */

void DWT_DelayMs(__IO uint32_t time)
{
	for(uint32_t i = 0; i < time; i++)
	{
		DWT_DelayUs(1000);
	}
}

 /**
  * @brief  DWT 计数实现精确延时，32位计数器
  * @param  time:延迟长度，单位1 s
  * @note   无
  * @retval 无
  */

void DWT_DelayS(__IO uint32_t time)
{
	for(uint32_t i = 0; i < time; i++)
	{
		DWT_DelayMs(1000);
	}
}

  /*****************************END OF FILE*******************************/
 ```

```c
#ifndef __BSP_DWT_H
#define __BSP_DWT_H

#include "stm32f10x.h"

/*
CYCCNT和us互相换算：
1/SystemCireClock*CYCCNT(s) = 1,000/SystemCoreClock*CYCCNT (MS) = 1,000,000/SystemCoreClock*CYCCNT (US) = (X)US

*/
#define DEMCR			 (*(volatile uint32_t*)(0xE000EDFC))
#define DWT_CTRL		 (*(volatile uint32_t*)(0xE0001000))
#define DWT_CYCCNT 	 (*(volatile uint32_t*)(0xE0001004))
 
#define DEMCR_TRCENA			(1<<24)
#define DWT_CTRL_CYCCNTENA		(1<<0)

void DWT_Init(void);
uint32_t DWT_GetTick(void);
uint32_t DWT_TickToMicrosecond(uint32_t tick,uint32_t frequency);
void DWT_DelayUs(__IO uint32_t time);
void DWT_DelayMs(__IO uint32_t time);
void DWT_DelayS(__IO uint32_t time);

#endif /*__BSP_DWT_H */
```
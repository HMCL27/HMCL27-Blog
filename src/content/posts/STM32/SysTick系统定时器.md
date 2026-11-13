---
title: "SysTick 系统定时器与精确延时"
published: 2026-08-09
description: "SysTick 的 24 位递减计数、重装载与中断配置，实现毫秒/微秒级延时。"
category: "STM32"
tags: [SysTick, 延时, Cortex-M3, 定时器]
draft: false
---

# SysTick 系统定时器与精确延时

Systick-系统定时器是属于CM3内核中的一个外设，内嵌在NVIC中，系统定时器是一个24bit的向下递减的计数器，计数器每计数一次的时间为1/SYSCLK，一般我们设置系统时钟SYSCLK等于72M。当重装载数值寄存器的值递减到0的时候，系统定时器就会产生一次中断，以此循环往复。


关于中断函数的学习
新建systick文件夹存放systick函数文件
bsp_systick_test.c
bsp_systick_test.h
可以作延时函数使用


//couter 减1的时间 等于 1/systick_clk
//当counter从 reload 的值减小到0的时候，为一个循环，如果开启了中断则执行中断服务程序
//同时 CTRL 的 countflag 位会置1
//这一个循环的时间为 reload * （1/systick_clk）

一段注释
```c
void SysTick_Handler(void)
{
}
```


此段函数会位于stm32f10x_it.c中

bsp_systick_test.c文件中需要此段函数
static uint64_t systick_count = 0;

static __INLINE uint32_t SysTick_Config(uint32_t ticks)

需要去 main.c 所链接的头文件 core_cm3.h 中把此段代码复制出来
注意返回值与直觉相反：初始化成功返回 0，失败返回 1（所以 `if(SysTick_Config(...)) { while(1); }` 的含义是失败时停在死循环）

去往与USER文件夹同级别的CMSIS文件夹内的system_stm32f10x.c文件内
复制系统的时钟频率代码 
SYSCLK_FREQ_72MHz
uint32_t SystemCoreClock = SYSCLK_FREQ_72MHz;   /*!< System Clock Frequency (Core Clock) */ 

其中的 SYSCLK_FREQ_72MHz
// 1ms 中断
SysTick_Config(SystemCoreClock / 1000);

// 1us 中断（微秒）
SysTick_Config(SystemCoreClock / 1000000);

// 10ms 中断
SysTick_Config(SystemCoreClock / 100);

// 100ms 中断
SysTick_Config(SystemCoreClock / 10);
重装载值 = SystemCoreClock / 1000 = 72,000,000 / 1000 = 72,000
计数频率 = SystemCoreClock = 72,000,000 Hz (不分频)
中断周期 = 72,000 / 72,000,000 = 0.001 秒 = 1ms

按这么计算，即计算频率为72MHz
每计数一次需要：1 / 72,000,000 秒 ≈ 13.89ns
计数 72,000 次需要：72,000 × 13.89ns ≈ 0.001秒 = 1ms

```c
void SysTick_DelayMsTest(uint32_t time)
{
  if(SysTick_Config(SystemCoreClock/1000))
  {
	  while(1);//初始化失败即一直循环在此
  }
  for(uint32_t i = 0;i<time;i++)
  {
	  while(!((SysTick->CTRL)&(1<<16)));
  
  }
}
```
时刻0ms (刚开始延时):
┌─────────────────────────────────────────┐
│ COUNTFLAG = 0                           │
│ (SysTick->CTRL) & (1<<16) = 0           │
│ !0 = 1 (真) → 继续在while循环中等待     │
└─────────────────────────────────────────┘

时刻0.5ms:
┌─────────────────────────────────────────┐
│ COUNTFLAG = 0 (还未到1ms)               │
│ 条件仍为真，继续等待                    │
└─────────────────────────────────────────┘

时刻1ms (到达):
┌─────────────────────────────────────────┐
│ COUNTFLAG 被硬件自动置为 1              │
│ (SysTick->CTRL) & (1<<16) = 非0值       │
│ !非0 = 0 (假) → 退出while循环           │
└─────────────────────────────────────────┘
           ↓
┌─────────────────────────────────────────┐
│ 退出while循环后，执行for循环体结束      │
│ i++ (i从0变为1)                         │
│ 检查 i < time 条件                       │
└─────────────────────────────────────────┘
           ↓
时刻1ms+ (进入下一轮):
┌─────────────────────────────────────────┐
│ 再次进入while循环                        │
│ 此时COUNTFLAG已被清除（读取后自动清除）  │
│ COUNTFLAG = 0                           │
│ 又开始等待下一个1ms                      │
└─────────────────────────────────────────┘


- `1 << 16` 表示将二进制1左移16位
    
- 结果：`0x00010000` 或二进制 `0000 0000 0000 0001 0000 0000 0000 0000`
    
- 这个值对应COUNTFLAG标志位的位置


|=      用来置1
&=~  用来复0

SysTick->CTRL  &= ~SysTick_CTRL_ENABLE_Msk;  //关闭定时器

//当counter 减1的时间 等于 1/systick_clk
//当counter 从 reload 的值减小到0的时候，为一个循环，如果开启了中断则执行中断服务程序
//同时 CTRL 的 countflag 位会置1
//这一个循环的时间为 reload * （1/systick_clk）

```c
/**
 * @brief  systick 查询寄存器延时函数接口
 * @param  time：延时时间，单位 ms
 * @note   使用时开启 SysTick 并配置为 1ms 周期
 * @retval 无
 */
void SysTick_DelayMsTest(uint32_t time)
{
  if(SysTick_Config(SystemCoreClock/1000))
  {
	  while(1);//初始化失败即一直循环在此
  }
  for(uint32_t i = 0;i<time;i++)
  {
	  while(!((SysTick->CTRL)&(1<<16)));
  
  }
  
   SysTick->CTRL  &= ~SysTick_CTRL_ENABLE_Msk;  //关闭定时器
  
}
最后结果


//获取系统计时器当前的值
uint64_t SysTick_GetCount(void)
{
    return systick_count;
}

//系统计时器加一
void SysTick_CountPlus(void)
{
    systick_count++;
}

```

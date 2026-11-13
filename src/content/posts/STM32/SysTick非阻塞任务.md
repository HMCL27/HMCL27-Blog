---
title: "SysTick 非阻塞任务调度"
published: 2026-08-10
description: "用 SysTick 周期中断驱动任务节拍，替代阻塞延时，让主循环保持响应。"
category: "STM32"
tags: [SysTick, 非阻塞, 任务调度, 状态机]
draft: false
---

# SysTick 非阻塞任务调度

利用Sysytick的循环周期性获取传感器信号以节省资源


首先在stm32f10x_it.c中写入 void SysTick_Handler(void)
// 在 SysTick_Handler 中（每 100ms 中断一次）
void SysTick_Handler(void)
{
    if(program_run_led_task.timer > 0x00)
    {
        program_run_led_task.timer--;  // 倒计时递减
        if(program_run_led_task.timer == 0)
        {
            program_run_led_task.flag = 1;  // 周期结束，触发翻转
        }
    }
}

在app_led.c中写入void Light_Task(void)，此为翻转LED，可以结合实际，如控制光敏传感器的测量周期
// 在 Light_Task 中（主循环）
void Light_Task(void)
{
    if(program_run_led_task.flag)  // 检查标志位
    {
        // 翻转 LED 状态
        LED_TOGGLE(...);
        program_run_led_task.flag = 0;  // 重置标志位
        program_run_led_task.timer = program_run_led_task.cycle;  // 重新计时
    }
}

每100ms读取一次光敏传感器的信号

可以建立main.h，在内部引用其它文件夹的内容，如与USER同级的APP文件夹

#ifndef __MAIN_H
#define __MAIN_H

#include "light/app_light.h"
#include "led/app_led.h"

#endif /* __MAIN_H */

现在实践非阻塞程序结构，实现led灯闪烁与光敏传感器同时工作

建立APP文件夹内存放led与light的应用层程序接口
此处可以起到类似全局变量的作用
 **可以像全局变量一样使用**，但需要：

1. 在头文件中用 `extern` 声明
2. 在源文件中实际定义（通常放在文件末尾）
3. 注意初始化时机和并发访问安全


```c
typedef struct
{
	uint32_t cycle;
	uint32_t timer;
	uint8_t flag;
}Led_TaskInfo;

extern Led_TaskInfo program_run_led_task ;

void ProgramRunLed_TaskInit(uint32_t program_run_led_task_cycle);
void ProgramRunLed_TaskReset(void);
void ProgramRunLed_Task(void);

/** app_led.c
  ******************************************************************************
  * @file       app_led.c
  * @author     embedfire
  * @version     V1.0
  * @date        2024
  * @brief      LED灯应用层功能接口
  ******************************************************************************
  * @attention
  *
  * 实验平台  ：野火 STM32F103C8T6-STM32开发板 
  * 论坛      ：http://www.firebbs.cn
  * 官网      ：https://embedfire.com/
  * 淘宝      ：https://yehuosm.tmall.com/
  *
  *****************************************************************************
  */
  
   #include "led/app_led.h"
   #include "led/bsp_gpio_led.h"
   
 Led_TaskInfo program_run_led_task = {0} ;
  
  /**
 * @brief  程序正常运行指示灯 计数复位
  * @param  无
  * @retval 无
  */
  
 void ProgramRunLed_TaskReset(void)
 {
	 program_run_led_task.timer = program_run_led_task.cycle;
	 program_run_led_task.flag  = 0;
 }
 
  /**
 * @brief  程序正常运行指示灯 任务初始化
  * @param  program_run_led_task_cycle：任务轮询周期，单位=100ms 节拍（与 SysTick 中断周期一致）
  * @retval 无
  */
 void ProgramRunLed_TaskInit(uint32_t program_run_led_task_cycle)
 {
	 program_run_led_task.cycle = program_run_led_task_cycle;
	 ProgramRunLed_TaskReset();
 }
 
   /**
 * @brief  程序正常运行指示灯 任务
  * @param  program_run_led_task_cycle：任务轮询周期，单位=100ms 节拍（与 SysTick 中断周期一致）
  * @retval 无
  */
 void ProgramRunLed_Task(void)
 {
	 if(program_run_led_task.flag)
	 {
		 LED_TOGGLE(G_LED_GPIO_PORT ,G_LED_GPIO_PIN);
		 ProgramRunLed_TaskReset();
	 }
 }
  /*******************************END OF FILE*********************************/
```
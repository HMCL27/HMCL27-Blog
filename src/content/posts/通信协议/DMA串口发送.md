---
title: "DMA 驱动 USART 发送"
published: 2026-08-25
description: "用 DMA1 通道 4 搬运 USART1 发送数据，CPU 只负责启动与进度统计。"
category: "通信协议"
tags: [DMA, USART, STM32F103, 数据传输]
draft: false
---

# DMA 驱动 USART 发送

关于DMA的使用

```c
#define SEND_BUF_SIZE 8200  //发送数据长度,最好等于sizeof(TEXT_TO_SEND)+2的整数倍.
u8 SendBuff[SEND_BUF_SIZE]; //发送数据缓冲区
MYDMA_Config(DMA1_Channel4,(u32)&USART1->DR,(u32)SendBuff,SEND_BUF_SIZE);//DMA1通道4,外设为串口1,存储器为SendBuff,长度SEND_BUF_SIZE.

j=sizeof(TEXT_TO_SEND);
    for(i=0;i<SEND_BUF_SIZE;i++)//填充数据到SendBuff
    {
        if(t>=j)//加入换行符
        {
            if(mask)
            {
                SendBuff[i]=0x0a;
                t=0;
            }else
            {
                SendBuff[i]=0x0d;
                mask++;
            }
        }else//复制TEXT_TO_SEND语句
        {
            mask=0;
            SendBuff[i]=TEXT_TO_SEND[t];
            t++;
        }
    }
    
   while(1)
    {
        t=KEY_Scan(0);
        if(t==KEY0_PRES)//KEY0按下
        {
            LCD_ShowString(30,150,200,16,16,"Start Transimit....");
            LCD_ShowString(30,170,200,16,16,"   %");//显示百分号
            printf("\r\nDMA DATA:\r\n");
          USART_DMACmd(USART1,USART_DMAReq_Tx,ENABLE); //使能串口1的DMA发送
            MYDMA_Enable(DMA1_Channel4);//开始一次DMA传输!
            //等待DMA传输完成，此时我们来做另外一些事，点灯
            //实际应用中，传输数据期间，可以执行另外的任务
            while(1)
            {
                if(DMA_GetFlagStatus(DMA1_FLAG_TC4)!=RESET) //判断通道4传输完成
                {
                    DMA_ClearFlag(DMA1_FLAG_TC4);//清除通道4传输完成标志
                    break;
            }
                pro=DMA_GetCurrDataCounter(DMA1_Channel4);//得到当前还剩余多少个数据
                pro=(SEND_BUF_SIZE-pro)*100/SEND_BUF_SIZE;//得到百分比（先乘后除，避免整除为0）
                LCD_ShowNum(30,170,pro,3,16);
            }              
            LCD_ShowNum(30,170,100,3,16);//显示100%
            LCD_ShowString(30,150,200,16,16,"Transimit Finished!");//提示传送完成
        }
        i++;
        delay_ms(10);
        if(i==20)
        {
            LED0=!LED0;//提示系统正在运行  
            i=0;
        }
    }
```

原理就跟先把东西存到箱子再直接把箱子用蛙港物流传输一样，玩家去弄别的事情，不用玩家亲自跑去送东西
目前来看要注意的是设定传输量的大小以及存入缓冲区的数据的大小，以及需要检查是否有数据没完全传输，和未传输的数据量大小
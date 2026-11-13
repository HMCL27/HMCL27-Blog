---
title: "IIC 驱动 AT24C02 EEPROM"
published: 2026-08-24
description: "软件 IIC 的起止、应答与时序，读写 AT24C02 并解释 0xA0/0xA1 的读写位。"
category: "通信协议"
tags: [IIC, AT24C02, EEPROM, 时序]
draft: false
---

# IIC 驱动 AT24C02 EEPROM

```c
#include "24cxx.h" 
#include "delay.h"
//初始化IIC接口
void AT24CXX_Init(void)
{
	IIC_Init();
}
//在AT24CXX指定地址读出一个数据
//ReadAddr:开始读数的地址  
//返回值  :读到的数据
u8 AT24CXX_ReadOneByte(u16 ReadAddr)
{
	u8 temp=0;
    IIC_Start();
	if(EE_TYPE>AT24C16)
	{
		IIC_Send_Byte(0XA0);	   //发送写命令
		IIC_Wait_Ack();
		IIC_Send_Byte(ReadAddr>>8);//发送高地址
		IIC_Wait_Ack();		 
	}else IIC_Send_Byte(0XA0+((ReadAddr/256)<<1));   //发送器件地址0XA0,写数据 	 

	IIC_Wait_Ack(); 
    IIC_Send_Byte(ReadAddr%256);   //发送低地址
	IIC_Wait_Ack();
	IIC_Start();
	IIC_Send_Byte(0XA1);           //进入接收模式			   
	IIC_Wait_Ack();	 
    temp=IIC_Read_Byte(0);
    IIC_Stop();//产生一个停止条件
	return temp;
}
//在AT24CXX指定地址写入一个数据
//WriteAddr  :写入数据的目的地址    
//DataToWrite:要写入的数据
void AT24CXX_WriteOneByte(u16 WriteAddr,u8 DataToWrite)
{
    IIC_Start();  
	if(EE_TYPE>AT24C16)
	{
		IIC_Send_Byte(0XA0);	    //发送写命令
		IIC_Wait_Ack();
		IIC_Send_Byte(WriteAddr>>8);//发送高地址
 	}else
	{
		IIC_Send_Byte(0XA0+((WriteAddr/256)<<1));   //发送器件地址0XA0,写数据 
	}	 
	IIC_Wait_Ack();	   
    IIC_Send_Byte(WriteAddr%256);   //发送低地址
	IIC_Wait_Ack();
	IIC_Send_Byte(DataToWrite);     //发送字节							   
	IIC_Wait_Ack();  		    	   
    IIC_Stop();//产生一个停止条件 
	delay_ms(10);	 
}
//在AT24CXX里面的指定地址开始写入长度为Len的数据
//该函数用于写入16bit或者32bit的数据.
//WriteAddr  :开始写入的地址  
//DataToWrite:数据数组首地址
//Len        :要写入数据的长度2,4
void AT24CXX_WriteLenByte(u16 WriteAddr,u32 DataToWrite,u8 Len)
{  	
	u8 t;
	for(t=0;t<Len;t++)
	{
		AT24CXX_WriteOneByte(WriteAddr+t,(DataToWrite>>(8*t))&0xff);
	}
}

//在AT24CXX里面的指定地址开始读出长度为Len的数据
//该函数用于读出16bit或者32bit的数据.
//ReadAddr   :开始读出的地址 
//返回值     :数据
//Len        :要读出数据的长度2,4
u32 AT24CXX_ReadLenByte(u16 ReadAddr,u8 Len)
{  	
	u8 t;
	u32 temp=0;
	for(t=0;t<Len;t++)
	{
		temp<<=8;
		temp+=AT24CXX_ReadOneByte(ReadAddr+Len-t-1); 	 				   
	}
	return temp;												    
}
//检查AT24CXX是否正常
//这里用了24XX的最后一个地址(255)来存储标志字.
//如果用其他24C系列,这个地址要修改
//返回1:检测失败
//返回0:检测成功
u8 AT24CXX_Check(void)
{
	u8 temp;
	temp=AT24CXX_ReadOneByte(255);//避免每次开机都写AT24CXX			   
	if(temp==0X55)return 0;		   
	else//排除第一次初始化的情况
	{
		AT24CXX_WriteOneByte(255,0X55);
	    temp=AT24CXX_ReadOneByte(255);	  
		if(temp==0X55)return 0;
	}
	return 1;											  
}

//在AT24CXX里面的指定地址开始读出指定个数的数据
//ReadAddr :开始读出的地址 对24c02为0~255
//pBuffer  :数据数组首地址
//NumToRead:要读出数据的个数
void AT24CXX_Read(u16 ReadAddr,u8 *pBuffer,u16 NumToRead)
{
	while(NumToRead)
	{
		*pBuffer++=AT24CXX_ReadOneByte(ReadAddr++);	
		NumToRead--;
	}
}  
//在AT24CXX里面的指定地址开始写入指定个数的数据
//WriteAddr :开始写入的地址 对24c02为0~255
//pBuffer   :数据数组首地址
//NumToWrite:要写入数据的个数
void AT24CXX_Write(u16 WriteAddr,u8 *pBuffer,u16 NumToWrite)
{
	while(NumToWrite--)
	{
		AT24CXX_WriteOneByte(WriteAddr,*pBuffer);
		WriteAddr++;
		pBuffer++;
	}
}
简单解释一下IIC的用法
```


您提供的代码是使用IIC（Inter-Integrated Circuit，也称为TWI）协议来与AT24CXX系列EEPROM芯片进行通信的驱动程序。

简单来说，IIC是一种常用的、双线制的串行总线通信协议，它允许一个主设备（Master，通常是微控制器MCU）控制多个从设备（Slave，这里就是EEPROM）。

根据您提供的代码，我将用流程化的方式解释一下IIC在读写数据时的基本用法。

### IIC 通信的基本概念

1.  **总线 (Bus):** 只有两条信号线：SCL（时钟线）和 SDA（数据线）。
2.  **主从模式:** MCU充当主设备，负责发起通信、控制时钟；EEPROM作为从设备，等待主设备的指令。
3.  **地址寻址:** 每个从设备都有一个 7 位地址（AT24C02 常用 `0x50`）。代码里的 `0xA0` / `0xA1` 不是两个设备地址，而是同一个 7 位地址左移一位后加读/写位：`0xA0` = 写，`0xA1` = 读。

---

### 核心操作流程解释

#### 1. 初始化 (Initialization)
*   **函数:** `AT24CXX_Init()`
*   **用法:** 在进行任何数据传输之前，必须先调用 `IIC_Init()`。这相当于配置MCU的GPIO引脚和内部时钟，使其能够正确地驱动SDA和SCL线。

#### 2. 写入数据 (Writing Data)
当您想把一个字节的数据写入EEPROM的某个地址时（例如使用 `AT24CXX_WriteOneByte`）：

1.  **开始传输:** 调用 `IIC_Start()`，产生一个起始条件信号。
2.  **发送设备地址和操作位:** 发送从设备的地址（如 `0XA0`），这个地址中包含了“写入”的标志位。
3.  **等待确认 (ACK):** 从设备接收到地址后会返回一个应答信号 (`IIC_Wait_Ack()`)，表示它准备好了。
4.  **发送目标地址:** 发送要写入数据的内存地址（高字节和低字节）。
5.  **等待确认:** 再次等待从设备的ACK。
6.  **发送数据:** 最后发送实际需要存储的数据 (`DataToWrite`)。
7.  **等待确认:** 等待最终的ACK。
8.  **结束传输:** 调用 `IIC_Stop()`，产生一个停止条件信号，完成本次通信。

#### 3. 读取数据 (Reading Data)
当您想从EEPROM的某个地址读取一个字节的数据时（例如使用 `AT24CXX_ReadOneByte`）：

1.  **设置读地址 (写入模式):** 首先，主设备需要告诉EEPROM：“请把内部指针移动到这个地址”。这与写入操作类似，发送地址和目标内存地址。
2.  **重复起始条件:** 在完成地址设置后，主设备会再次发起一个“重复起始条件” (`IIC_Start()`)。这是关键步骤，它告诉EEPROM：“好了，现在我要开始读取了。”
3.  **切换到读模式:** 发送从设备的地址，但这次的地址中包含了“读取”的标志位（如 `0XA1`）。
4.  **等待确认 (ACK):** 等待从设备确认。
5.  **接收数据:** 调用 `IIC_Read_Byte(0)` 从总线上接收到EEPROM存储的数据。
6.  **结束传输:** 调用 `IIC_Stop()`，完成本次通信。

### 总结表格

| 操作      | 目的          | IIC关键步骤                                                                                                    | 代码对应函数示例               |
| :------ | :---------- | :--------------------------------------------------------------------------------------------------------- | :--------------------- |
| **初始化** | 配置硬件接口      | 设置时钟和引脚                                                                                                    | `AT24CXX_Init()`       |
| **写入**  | 将数据存入EEPROM | Start $\rightarrow$ 发送地址(W) $\rightarrow$ 发送地址 $\rightarrow$ 发送数据 $\rightarrow$ Stop                       | `AT24CXX_WriteOneByte` |
| **读取**  | 从EEPROM取出数据 | Setup Address (W) $\rightarrow$ Repeated Start $\rightarrow$ 发送地址(R) $\rightarrow$ 接收数据 $\rightarrow$ Stop | `AT24CXX_ReadOneByte`  |

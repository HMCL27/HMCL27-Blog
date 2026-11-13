---
title: "MQTT 与 ESP8266 收发实践"
published: 2026-10-11
description: "ESP8266 订阅与发布 MQTT 主题，用 cJSON 组装和解析上报报文。"
category: "物联网平台"
tags: [MQTT, ESP8266, AT指令, cJSON]
draft: false
---

# MQTT 与 ESP8266 收发实践

如果需要接收信息，必须订阅主题
```c
configureMQTTSub(SubTime);//订阅 SubTime
```
此函数需要在初始化时候执行，即只执行一次即可
后续需要在

```c
SubMQtt_IS("SubTime","temp",temp,"time",time_Buf); //接收主题 SubTime 并处理数据
```

```c
//==========================================================
//	函数名称：	SubMQtt_II
//	函数功能：	MQTT数据发送两个数字（含主题指定）
//==========================================================
_Bool SubMQtt_II(const char* Topic ,
    const char* key0, int Int_value0, 
    const char* key1, int Int_value1)
{
    // 创建JSON对象
    cJSON *root = cJSON_CreateObject();
    if(!root) {
        UsartPrintf(USART1, "Create JSON failed!\r\n");
        return 0;
    }
    // 添加传感器数据（参数化）
    cJSON_AddNumberToObject(root,key0,Int_value0);
    cJSON_AddNumberToObject(root,key1,Int_value1);
    // 生成JSON字符串
    char *jsonRaw = cJSON_PrintUnformatted(root);
    cJSON_Delete(root);
    if(!jsonRaw) {
        UsartPrintf(USART1, "JSON print failed!\r\n");
        return 0;
    }

    // 转义双引号
    char *jsonEscaped = EscapeJSONForMQTT(jsonRaw);
    free(jsonRaw);
    if(!jsonEscaped) {
        UsartPrintf(USART1, "Escape failed!\r\n");
        return 0;
    }

    // 检查总指令长度
    char cmd[256];
    int cmdLen = snprintf(cmd, sizeof(cmd), 
        "AT+MQTTPUB=0,\"%s\",\"%s\",1,0\r\n", Topic,
        jsonEscaped);
    
    if(cmdLen >= sizeof(cmd)) {
        UsartPrintf(USART1, "CMD too long:%d/%d\r\n", cmdLen, sizeof(cmd));
        free(jsonEscaped);
        return 0;
    }

    // 调试输出
    UsartPrintf(USART1, "[MQTT CMD] %s\r\n", cmd);

    // 发送指令
    _Bool result = 0;
    // 注意：ESP8266_SendCmd 内部有超时等待，长时间关中断会冻结 SysTick、丢串口数据；
    // 若工程里用了 FreeRTOS 或中断收发，应改用互斥量/短临界区，不要整段关中断。
    __disable_irq();
    result = ESP8266_SendCmd(cmd, "OK"); // 使用原有发送函数
    __enable_irq();

    free(jsonEscaped);
    return result;
}
	
```

关于修改输出函数，只需要把其中关键部分修改即可，如key0 key1以及
```c
cJSON_AddNumberToObject
```
可以改成
```c
cJSON_AddStringToObject
```
以处理字符串
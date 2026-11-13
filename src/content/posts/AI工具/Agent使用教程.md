---
title: "Agent 使用教程：API、CC Switch 与 Claude Code"
published: 2026-10-02
description: "从日日新、DeepSeek 的 API Key 到 CC Switch 与 Claude Code 接入编译器的完整流程。"
category: "AI 工具"
tags: [AI Agent, Claude Code, API, 工具链]
draft: false
---

# Agent 使用教程：API、CC Switch 与 Claude Code

## 1. 日日新商汤 API
### 1.1 首先进入官网注册账号

https://www.sensenova.cn/

![日日新01](/img/posts/2026/10/02/Agent/日日新01.png)

![日日新02](/img/posts/2026/10/02/Agent/日日新02.png)

### 1.2 进入控制台

![日日新03](/img/posts/2026/10/02/Agent/日日新03.png)

### 1.3 新建API Key

![日日新04](/img/posts/2026/10/02/Agent/日日新04.png)

![日日新05](/img/posts/2026/10/02/Agent/日日新05.png)

同理DEEPSEEK之类的网站亦可以创建API

这里大概意思是通用积分池给大部分模型用，每5个小时可以用60.000额度，每7天可以用600.000额度，专用积分池只能给特定模型用。最好用的时候看着点，跑一半断了可能前功尽弃。
## 2. CC Switch
### 2.1 下载CC switch

https://cc-switch.cc/

![CCSwitch01](/img/posts/2026/10/02/Agent/CCSwitch01.png)

![CCSwitch02](/img/posts/2026/10/02/Agent/CCSwitch02.png)

### 2.2 安装CC Switch

![CCSwitch03](/img/posts/2026/10/02/Agent/CCSwitch03.png)

![CCSwitch04](/img/posts/2026/10/02/Agent/CCSwitch04.png)

![CCSwitch05](/img/posts/2026/10/02/Agent/CCSwitch05.png)
这里建议不要类似 D:\CC Switch 一样直接放根目录
D:\AI\CC Switch 这个路径更好，后续DSH，豆包工作文件夹也可以塞进去，方便整理

![CCSwitch06](/img/posts/2026/10/02/Agent/CCSwitch06.png)

![CCSwitch07](/img/posts/2026/10/02/Agent/CCSwitch07.png)

### 2.3 配置CC Switch

### 2.4 先开启路由，有部分 API 需要启用
![CCSwitch08](/img/posts/2026/10/02/Agent/CCSwitch08.png)

![CCSwitch09](/img/posts/2026/10/02/Agent/CCSwitch09.png)

## 3. 使用 Claude Code
### 3.1 用CC Switch下载Claude Code

![ClaudeCode01](/img/posts/2026/10/02/Agent/ClaudeCode01.png)

直接一个CC Switch一步到位，要官方下载方式也行

https://code.claude.com/docs/zh-CN/quickstart

### 3.2 将Claude Code接入到各类编译器

#### VS Code
这里直接跳过下载与初始设置，B站自己看教程

https://code.visualstudio.com/

![ClaudeCode02](/img/posts/2026/10/02/Agent/ClaudeCode02.png)

这个是将Claude Code 接入到VS Code的官方扩展插件

使用示例

![ClaudeCode03](/img/posts/2026/10/02/Agent/ClaudeCode03.png)


#### PyCharm
这个我也是第一次安装，顺便写一遍完整教程吧

https://www.jetbrains.com/zh-cn/pycharm/

![PyCharm01](/img/posts/2026/10/02/Agent/PyCharm01.png)

![PyCharm02](/img/posts/2026/10/02/Agent/PyCharm02.png)

![PyCharm03](/img/posts/2026/10/02/Agent/PyCharm03.png)

![PyCharm04](/img/posts/2026/10/02/Agent/PyCharm04.png)

![PyCharm05](/img/posts/2026/10/02/Agent/PyCharm05.png)

这里上面的CC GUI是国产玩意，更适合中国宝宝体质，下面那个纯命令行的官方插件没必要了

![PyCharm06](/img/posts/2026/10/02/Agent/PyCharm06.png)

![PyCharm07](/img/posts/2026/10/02/Agent/PyCharm07.png)

![PyCharm08](/img/posts/2026/10/02/Agent/PyCharm08.png)

这里是为了和CC Switch同步

![PyCharm09](/img/posts/2026/10/02/Agent/PyCharm09.png)

PyCharm配置到这里就可以使用了

这里再加个读文件的 XLS Loader 插件

![PyCharm10](/img/posts/2026/10/02/Agent/PyCharm10.png)

![PyCharm11](/img/posts/2026/10/02/Agent/PyCharm11.png)

![PyCharm12 1](/img/posts/2026/10/02/Agent/PyCharm12%201.png)

这是发送文件的用法之一，直接复制地址给它应该也能读，感觉没VS Code好用就是

## 4. 豆包
https://www.doubao.com/

### 4.1 下载并使用豆包电脑版
![DouBao01](/img/posts/2026/10/02/Agent/DouBao01.png)

![DouBao02](/img/posts/2026/10/02/Agent/DouBao02.png)

这里主要使用工作模式，工作模式选择本地电脑会调用本地资源，云端适合手机使用，比如让他设计PPT并发给你。

选择指定的文件夹会优先查看该文件夹下的资源，比如单片机的资料和代码，这样不必全盘查或者上网慢慢查，节省时间。也可以直接给它本地文件地址或者网络链接试试。

需要注意的是你在把代码或者重要文件交给它前必须**备份**，按需要也可以告诉它备份位置，如它改不回来的话。

![DouBao03](/img/posts/2026/10/02/Agent/DouBao03.png)

![DouBao04](/img/posts/2026/10/02/Agent/DouBao04.png)

可以关注一波有没有活动，额度没了可就用不了咯

## 5. DeepSeek
### 5.1 先注册个号

https://platform.deepseek.com/sign_in

![DeepSeek01](/img/posts/2026/10/02/Agent/DeepSeek01.png)

### 5.2 模型细节

价目表

|                         |      | DeepSeek-V4.1-Flash | DeepSeek-V4-Pro-0813 |
| ----------------------- | ---- | ------------------- | -------------------- |
| 百万tokens输入  <br>（缓存命中）  | 空闲时段 | 0.02元               | 0.15元                |
| 百万tokens输入  <br>（缓存未命中） | 空闲时段 | 1元                  | 4.5元                 |
| 百万tokens输出              | 空闲时段 | 4元                  | 13.5元                |

(1) 模型名请使用 `deepseek-flash`。旧模型名 `deepseek-v4-flash`、`deepseek-v4-flash-vision-exp` 仍可调用，但对应模型已下线，请求将由 DeepSeek-V4.1-Flash 模型提供服务，并按 Flash 价格计费。

(2) 空闲时段价格为高峰时段价格的一半。北京时间周一至周五（不含中国法定节假日）9:00 - 12:00、14:00 - 18:00 为高峰时段；其余时段，包括周末及中国法定节假日全天均为空闲时段。

(3) 更多并发限制细节，请参考[限速与隔离](https://api-docs.deepseek.com/zh-cn/quick_start/rate_limit)。

最好在低谷时段用，不然一下额度就没咯

### 5.3 使用 DSH 蓝色大肥鱼
https://www.dshdesktop.cn/

![DeepSeek02](/img/posts/2026/10/02/Agent/DeepSeek02.png)

安装的细节不写了，反正后续都可以改。

![DeepSeek03](/img/posts/2026/10/02/Agent/DeepSeek03.png)

![DeepSeek04](/img/posts/2026/10/02/Agent/DeepSeek04.png)


![DeepSeek05](/img/posts/2026/10/02/Agent/DeepSeek05.png)

这里自定义提供方亦可以接入日日新，具体的申请地址可以看文档也可以看示例，但是要简单修改一下
https://token.sensenova.cn/v1
申请会自动接入后缀，这里留到v1就行

![DeepSeek06](/img/posts/2026/10/02/Agent/DeepSeek06.png)

![DeepSeek07](/img/posts/2026/10/02/Agent/DeepSeek07.png)

具体使用方式和豆包工作模式差不多，也可以修改本地文件，还有插件市场可以美化和增加功能
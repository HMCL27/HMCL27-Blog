---
title: "本地 AI 部署：LM Studio 与 DeepSeek"
published: 2026-10-03
description: "在 Windows 上用 LM Studio 下载、调试本地模型，并接入 CC Switch 的实践记录。"
category: "AI 工具"
tags: [本地大模型, LM Studio, DeepSeek, 模型部署]
draft: false
---

# LM Studio

## 1.1 下载LM Studio

https://lmstudio.ai/download

![LMStudio01](/img/posts/2026/11/08/本地AI-LMStudio/LMStudio01.png)

## 1.2 安装LM Studio

![LMStudio02](/img/posts/2026/11/08/本地AI-LMStudio/LMStudio02.png)

![LMStudio03](/img/posts/2026/11/08/本地AI-LMStudio/LMStudio03.png)

![LMStudio04](/img/posts/2026/11/08/本地AI-LMStudio/LMStudio04.png)

安装好后会推荐你下一个模型，可下可不下

## 1.3 设置LM Studio

![LMStudio05](/img/posts/2026/11/08/本地AI-LMStudio/LMStudio05.png)

![LMStudio06](/img/posts/2026/11/08/本地AI-LMStudio/LMStudio06.png)

## 1.4 下载本地模型

下面点主界面左侧边栏的小机器人头像进入模型下载界面，这里有很多开源模型
选择最多点赞或者最多下载都行，可以看到流行的模型。
一般27B 14B 这种数字带个B的，数字越大越聪明。
要是后面带神秘后缀，或者前缀，比如E4B,E2B 就代表做了一些优化，数字变小适合在低配机器上运行。

![LMStudio07](/img/posts/2026/11/08/本地AI-LMStudio/LMStudio07.png)

那么怎么看自己电脑是否适合运行这些模型呢，主要是内存与显存。

![LMStudio08](/img/posts/2026/11/08/本地AI-LMStudio/LMStudio08.png)

![LMStudio09](/img/posts/2026/11/08/本地AI-LMStudio/LMStudio09.png)

你可以看到我的GPU有两个，后面有个小开关，打开意味调用它。
以上就是软件内的系统信息，但是这个不够

![LMStudio10](/img/posts/2026/11/08/本地AI-LMStudio/LMStudio10.png)

右键任务栏空白处打开任务管理器，查看当前内存显存使用量。
假如我把编程时候的软件全部打开，占用15个G，我的显存占用2G，那么剩下15g内存，9G显存就是我能用来跑AI模型的家底。

## 1.5 调试设置

![LMStudio11](/img/posts/2026/11/08/本地AI-LMStudio/LMStudio11.png)

这是运行AI 的运行库之类的，Vulkan是显卡通用运行库，CPU只能CPU运行，CUDA只能英伟达显卡运行

![LMStudio12](/img/posts/2026/11/08/本地AI-LMStudio/LMStudio12.png)

如果我改成CUDA的话我的显卡只显示英伟达的P106，Vulkan才能两张都看到
具体哪个好看电脑，自己慢慢测试得了

![LMStudio13](/img/posts/2026/11/08/本地AI-LMStudio/LMStudio13.png)

回到这里，刚才我给了我的系统信息，如果我打算用P106运行大模型的话，我有6G显存，这个模型只有4.8G，装得下，那么就下载先。

![LMStudio14](/img/posts/2026/11/08/本地AI-LMStudio/LMStudio14.png)

这里显示下载的模型进度，有时候比较慢，挂后台慢慢下。

![LMStudio15](/img/posts/2026/11/08/本地AI-LMStudio/LMStudio15.png)

接下来开始细调参数

## 1.6 测试模型速度

![LMStudio16](/img/posts/2026/11/08/本地AI-LMStudio/LMStudio16.png)

这里左下角点记住此次更改对于此模型
上下文长度就是一次运行中能回忆的内容长度与输出内容的长度，改这里会更改右上角显示的模型总大小，我只有6G显存，留一点点控制5.2G防止系统崩溃。这是纯GPU运行的情况，

如果内存很多比如我有15G剩余内存，那么总大小可以控制在20.2G以内。这时候右上角左边GPU的大小要控制5G左右，通过跳针GPU卸载来控制。代价就是一部分运算交给CPU，可能会慢很多。

![LMStudio17](/img/posts/2026/11/08/本地AI-LMStudio/LMStudio17.png)

既然如此我就简单做个测试，看看两种设置下的输出速度区别

![LMStudio18](/img/posts/2026/11/08/本地AI-LMStudio/LMStudio18.png)

刚加载完，这里显示显存占用不像我设置里的5G，只有3.7G，模型优化相关的事了，有的会一致

![LMStudio19](/img/posts/2026/11/08/本地AI-LMStudio/LMStudio19.png)

这是一部分给CPU的，虽然CPU负责的不多

![LMStudio20](/img/posts/2026/11/08/本地AI-LMStudio/LMStudio20.png)

![LMStudio21](/img/posts/2026/11/08/本地AI-LMStudio/LMStudio21.png)

在阅读文件与输出速度上均有提升

## 1.7 接入CC Switch

![LMStudio22](/img/posts/2026/11/08/本地AI-LMStudio/LMStudio22.png)

控制台下面可以输出日志，要是有问题可以复制日志丢给别的AI解决

这里可以设置公开端口，这样别人也能用你的电脑上的AI，同时可以接入CC Switch来使用，但是一般这种小模型不会调用CLI工具，也就不能直接改你代码，还不如只在这里聊聊天

![LMStudio23](/img/posts/2026/11/08/本地AI-LMStudio/LMStudio23.png)

API协议要选择anthropic-messages

![LMStudio24](/img/posts/2026/11/08/本地AI-LMStudio/LMStudio24.png)

打开 CC Switch 的高级选项点获取模型，有反应就成功了

同理，DSH亦可这样设置

![LMStudio25](/img/posts/2026/11/08/本地AI-LMStudio/LMStudio25.png)

这里的API协议要选择anthropic-messages

![LMStudio26](/img/posts/2026/11/08/本地AI-LMStudio/LMStudio26.png)

需要注意的是，当你开放端口给别的软件用时，这里会自动加载软件里启用的模型，如果你选择了A后又启用B，这里可能不会自动卸载，需要手动卸载

![LMStudio27](/img/posts/2026/11/08/本地AI-LMStudio/LMStudio27.png)

启用这个应该可以自动卸载，最好感觉很慢很慢时候来看一眼
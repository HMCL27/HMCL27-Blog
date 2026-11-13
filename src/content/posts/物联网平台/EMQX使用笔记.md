---
title: "EMQX 消息服务器使用笔记"
published: 2026-10-10
description: "在 Windows 安装 EMQX 服务、启停与常用管理命令。"
category: "物联网平台"
tags: [EMQX, MQTT, 消息代理, iStoreOS]
draft: false
---

# EMQX 消息服务器使用笔记

```c
cd C:\aHMCL\Tool\EMQX\emqx-5.3.0-windows-amd64\bin //EMQX服务文件位置
.\emqx.cmd install     //安装EMQX服务
.\emqx.cmd uninstall   //卸载EMQX服务
.\emqx.cmd console      //启动EMQX服务
```
此为EMQX服务的安装路径与使用指令，不用的时候就该删了

用 `console` 方式启动时是前台运行，关闭终端服务就停了；用 `install` 注册成 Windows 服务则开机自启、关闭终端不影响
我的软路由上也安装了EMQX，不过看着这两差不多，功能应该通用的

EMQX负责信息的转发

由于此服务的一个程序占用大量CPU计算，直接卸载得了，平时使用软路由的服务
---
layout: post
title: XP 下无法获取网络适配器列表
date: 2011-10-17 09:10:57
categories: os
tags: registry, windows
---

今天发现备份和测试用的 XP 机的网络忽然不行了，进入控制面板的网络连接，发现里面什么都没有；一刷新，却出现这样的信息：

> The network connections folder was unable to retrieve the list of Network adapters on your machine. Please make sure that the Network Connections service is enabled and running.

运行 `services.msc`，发现 Network Connections 果然没有起来；可是，启动之后，也没有发现问题好转。

于是在网上查，遇到这样的问题，一般第一想到的是注册表，于是找了个注册表的解决方案：

把 `HKEY_LOCAL_MACHINESYSTEMCurrentControlSetControlNetwork` 下的 `Config` 删除，重启之后会重新创建，试了，也无用。

最后试了一个比较简单的方法，CMD 下依次输入：

```bash
regsvr32 netshell.dll
regsvr32 netcfgx.dll
regsvr32 netman.dll
```

每条命令都运行成功，重启之后...好了！吉人天相！

可是不明白，好端端的，为什么 windows 老跟我过不去...
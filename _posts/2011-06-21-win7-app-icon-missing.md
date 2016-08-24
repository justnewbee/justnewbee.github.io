---
layout: post
title: Win7 软件图标丢失
date: 2011-06-21 11:06:59
categories: os
tags: windows
---

我已经碰到过好多次了，今天又碰到了一次，就是 Win7 下某些软件的图标会突然间变成一个「破」图标。但其实该软件还健在，且你从图标属性里面看去，也是正常的。

这肯定是 bug，而且我知道是 Window 文件系统中的一个叫做什么 iconcache 文件导致的。以前有解决过，但是不太记得如何搞定的，所以在这里记录一下。

在 CMD 里依次输入以下命令（最后一个是重启）：

1. `taskkill /IM explorer.exe /F`
2. `CD /d %userprofile%AppDataLocal`
3. `DEL IconCache.db /a`
4. `shutdown /r`

性急了，照着就做了，结果当然是 OK 的。不过，我觉得手工删除 `IconCache` 这个罪魁，然后重启 explorer 应该也是可以的。
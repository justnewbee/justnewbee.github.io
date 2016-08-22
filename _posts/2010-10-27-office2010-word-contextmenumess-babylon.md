---
layout: post
title: Office2010 Word 右键菜单弹出即消失
date: 2010-10-27 09:10:01
tags:  software
---

今天写文档的时候，突然间发现 Word 2010 的右键菜单有问题，出来闪一下就消失。之前肯定是好的，至少昨天是好的。

以为是 Esc 键或者哪颗键卡住了，敲了半天，一想，不会啊，Firefox 没问题，桌面没问题...

那么会不会是什么运行程序跟它冲突了呢。看了半天进程，没有可疑。注销，重启，都一样。

会不会整个 Office 都这样呢？结果只有 Word...

看来重装也没用了...

最近对电脑有什么改动？想来想去只有 Babylon 了。它很可疑，为什么？因为它是翻译软件。但是我没有运行着它啊，难不成...插件？！

果然在 Word 的 COM Add-ins 里面被我找到打者勾的 Babylon Translation Addin：

![](/images/posts/word2010_babylon.png)

去掉，果然 OK 了就。
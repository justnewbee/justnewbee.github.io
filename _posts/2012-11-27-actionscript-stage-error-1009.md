---
layout: post
title: ActionScript stage 属性 Error #1009
date: 2012-11-27 09:11:07
categories: frontend
tags: as, upload
---

好久没有更新 FlashUpload 的代码了，没想到一直藏着个问题，而且挺严重。我的 debug 版的 flash player 一加载就弹一个错误框，大致内容是如下：

> Error #1009：Cannot access a property or method of a null object reference

看了一下代码行数，刚好是 `stage.align = StageAlign.TOP_LEFT;`，猜是 `stage` 的问题，果不其然，在 [这里](http://stackoverflow.com/questions/3641710/cannot-access-a-property-or-method-of-a-null-object-reference) 找到答案，解决了。

##【补充】

这个应该是 Adobe Flash Player Debug 版最近出的问题，在 swfupload 的官方 demo 上也弹了错误框，

> TypeError: Error #1009: Cannot access a property or method of a null object reference.
at SWFUpload()

在 github 上的那个 flash 剪贴板也有类似的问题：

> TypeError: Error #1009: Cannot access a property or method of a null object reference.
> at Clippy$/main()
> at MethodInfo-65()
> at flash::Boot()
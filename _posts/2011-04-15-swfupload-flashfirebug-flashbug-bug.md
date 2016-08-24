---
layout: post
title: SWFUpload, FlashFirebug, Flashbug 的 bug
date: 2011-04-15 10:04:25
categories: software
tags: as
---

一直以来只在**用** `SWFUpload` 这个组件，即使有些改动也只是在 JS 层上，但是最近发现了一个十分严重但完全又不严重的 bug。

为什么这么说呢：

* 「十分严重」是指它能让浏览器崩溃
* 「完全不严重」 呵呵，它只在 IE 下出现

但总不能不解决...

这个 bug 出现在 flash 对象销毁的时候，不管是通过 SWFUpload 提供的 `destroy()` 方法还是简单的把 flash 节点或者它的父节点删除掉。

研究了一天，发现这个 bug 有以下的规律：

![](/images/posts/swfupload_destroy_bug.png)

看来不得不攻入 SWFUpload 的核心 - AS 了。我大言不惭地可以号称 JS 有点入门，但是 AS...没得办法，只有硬着头皮上。

我的开发模式很土，
用 [FDT](http://fdt.powerflasher.com/) 写代码（嘿嘿 我 Eclipse 也挺入门的），不会 build... build 用 [FlashDevelop](http://www.flashdevelop.org/)。

习惯 JS 中的 `console`，其实 AS 里通过 `ExternalCall` 也可以调用，但是最方便估计还是 `trace` 吧 - 目前只会它。好在 Firebug 有插件可以直接看 Trace。Mozilla 网站上有两个 Flash 插件 - Flashbug 和 FlashFirebug：

* Flashbug 会给 Firebug 加入 Shared Objects 和 Flash Console 两个 Tab，并扩展 Net Tab
* FlashFirebug 会加一个 Flash Inspector 图标以及一个 Flash Tab

FlashFirebug 有 一个 bug - 当它的 Flash Tab 是启动状态的时候，会把页面上的 Flash 的 style 属性改成「visibitly: visible !important;」。这对于我的 SWFUpload 是要命的，因为我需要给 Flash 的样式加上 `position: absolute` 等.

Flashbug 也有 bug，我本来禁用了 FlashFirebug 想只用 Flashbug，结果发现 trace 信息都没有打印出来。

所以，目前是这个状态：

FlashFirebug 装了，但它的 Flash Tab 是禁用的；Flashbug 装了，它的 Shared Objects 是禁用的 - 因为暂时用不上。

奶奶的，老子也算是个 Flash 开发了。
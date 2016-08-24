---
layout: post
title: Eclipse 显示 GC
date: 2010-09-09 04:09:03
categories: software
tags: eclipse
---

记得很早以前，3.0 或者 3.1 吧，那时候的 Eclipse 右下角有个 GC，显示用掉了多少内存，你可以随时手动做一下 GC。十分 miss 这个 feature，在想，这么好的 feature 怎么他们就忍心丢掉了呢...

今天终于发现原来他们没有丢掉，而是默认关掉了，在 `Preference > General` 下设置一下即可：

![](/images/posts/eclipse_pref_gc.png)
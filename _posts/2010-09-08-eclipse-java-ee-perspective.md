---
layout: post
title: Eclipse 在 Java EE Perspective 下文件的排序问题
date: 2010-09-08 14:09:58
tags: software
---

一直以来都用 Eclipse 的「Java Perspective」，并不是因为喜欢，其实我更喜欢「Java EE Perspective」，它在 `src` 及依赖包的显示上更为简明，唯一的缺陷是它的文件和目录排序有问题，我是一直这么认为的...

由于项目的架构问题，有很多的 `src` 目录，导致 Project Explorer 在 Java Perspective 下拉得很长，于是我切到 Java EE Perspective，本想看看就能习惯了，然而，实在是掌控不了自己的心魔...

文件与目录同等对待，仅按照字母排序，这样的排序太叫人恶心了：

![](/images/posts/eclipse_jeeperspective_disordered.png)

因为 Java Perspective 从来没这个问题，所以一直以来都以为这个是 By Design 的。今天突发奇想，怀疑是否是某个插件导致的，于是先到同事的机子上试，果然他的是没问题的。然后搞了个全新的 Helios，也果然。

看来八成是哪个插件造的孽了。

首先，我是一个喜欢倒腾软件的人，尤其喜欢倒腾有插件的软件，工作机装了这些插件：

* checkstyle
* easyexplore
* eclemma
* findbugs
* grepconsole
* jautodoc
* jbosstools
* jseclipse
* jslex
* pmd
* m2e
* spket
* zipeditor

好在从来都是用链接的形式安装插件的，最近改成直接把插件丢到 `dropin` 目录，都是很容易卸载的方式，可以一个个卸了试试。

原本怀疑是 JBoss Tools，它看来嫌疑最大，因为它最大，是个典型的 EE 工具组合，试验发现错怪好人了。

在公司试验时因为一时心急，全部先卸了，完了再全部放回去的时候，发现好了...可是再稍微折腾一下，又坏了，坏了之后就一直坏了。看来是有触发因子的。回到家继续试，自己的机器插件稍微少了几个：

* easyexplore
* grepconsole
* jbosstools
* jseclipse
* jslex
* m2e
* spket
* zipeditor

所以很耐心地一个一个试过来，最后发现卸载了 `m2e` 之后就好了，估计就是它了。我不能确定是否是某几个插件合起来造的孽，但 m2e 应该是脱不了干系了。
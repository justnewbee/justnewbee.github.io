---
layout: post
title: Safari 调试工具 Oh YeaH
date: 2010-06-12 03:06:58
categories: software
tags: software
---

一直以来认为 Safari 在 Web 调试上是个残疾直到今天 才发现一直错怪了它... 而且... 它用的居然和 Chrome 是一个调试工具 - Web Inspector，只是 Chrome 在这点上做的比 Safari 好，一开始就把 Web Inspector 给 Enable 了，而且做了一些扩展，并给了它另外一个名字 - Develop Tools。

而 Safari 把它雪藏的太深了，以至于像我这样会玩软件设置的人都...

## 如何启用

Safari Prefernces --&gt; Advanced 最下面的选项：

![](/images/posts/safari_preferrence_advanced.png)


勾选，然后你就可以在右键菜单上看到「Inspect Element」选项了，跟 Chrome 一模一样：

![](/images/posts/safari_context_menu.png)

## 界面

来看看两个的界面。

### Safari Web Inspector

![](/images/posts/safari_web_inspector.png)

### Chrome Develop Tools

![](/images/posts/chrome_devtool.png)

界面看起来几乎一样，本来就是一个东西么。区别就在 Chrome 多了一些功能，再看仔细一点，Safari 上面的那个标尺，更圆润一点。猜是 Safari 对 HTML5 Vanvas 的支持更强大一点，毕竟人家是Canvas的发明者啊。

## YeaH
在 Safari 上调 CSS 和 JS，没问题啦！

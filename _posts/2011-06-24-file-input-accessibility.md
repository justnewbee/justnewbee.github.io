---
layout: post
title: File Input Accessibility 研究
date: 2011-06-24 05:06:10
tags: develop, javascript
---

最近一直被用户可访问性的 bug 缠着，QA 要求当光标 tab 到我的上传控件上的时候，按「Enter」键能让文件浏览框弹（且叫它 file dialog 吧）出来。

可是上传靠的是 flash 或者 file input。

flash，虽然我是个 NB 的 flash 开发，可是 NB == NewBie...

先拣软的啃吧，所以，今天稍稍对 file input 的特性做了一下研究。

首先来看看 file input 在不同浏览器上的特性。

我测试的浏览器有：

Firefox 5（今早刚更新了的 魔贼辣已经开始主推 5 了，据我的经验，5 下怎样，3 和 4 不会有太大区别）、IE9、Opera 11、Chrome 10、Safari 5。

## 外形

![](/images/posts/fileinput_appearance_in_browsers.png);

Firefox、IE 和 Opera 一路，左边 text 框，右边按钮，当然是 `dir` 为默认的 `ltr` 的时候。他们之间稍有区别，IE 下 `dir` 和 `multiple` 无效；Firefox、IE 的按钮永远是「Browse...」；
Opera 在 `multiple` 的时候是「Add files」，非 `multiple` 的时候是「Choose...」。

Chrome 和 Safari 肯定一路，左边「Choose File」按钮（即使设置 `multiple` 也是这样），右边「No file chosen」的文本（`dir` 可以改变位置）。

## 鼠标点击

虽然样子不太一样，但都是由两部分组成，按钮 + 非按钮。除了 IE，其余的浏览器把两者看成一体，不论点击哪个，都会出 file dialog；IE 只对点击按钮有反应。

## Tab聚焦

页面上放两个 `input`，一个 text，一个 file。先光标挪到 text 上，按「tab」。

```
<input type="text" /><input type="file" />
```

Firefox 直接聚焦到按钮；Chrome 和 Safari 把 file input 当成一个整体聚焦；IE 分两次聚焦，先文本框后按钮；Opera 一次聚焦，但必须两次「Tab」，经过文本后才能对按钮聚焦。

## 聚焦后键盘操作 空格和回车

Firefox、Chrome、Safari 聚焦后均可对「空格」和「回车」响应；IE8/9 两次聚焦均能响应「空格」，但「回车」无效，但 IE6/7 在文本框上按「空格」是输入；Opera 都能响应，但必须在按钮聚焦（按两次「tab」）后。

## 屏幕阅读器

使用 Thunder 作为屏幕阅读器，「tab」键看能否被阅读。

结果只有 Firefox 和 Safari 可以被读到，IE 在有 label 的时候能读到，两次聚焦都会读 label 的文本，第二次会加上 "button"。

## 脚本 focus 后空格或回车

在页面加载后对 file input 执行 `focus()`，结果是所有的浏览器都可以聚焦。除了 IE6/7 无视空格和回车之外，其他浏览器都可以正确响应 - 因为 IE 天生对回车无视，而 IE6/7 的 text 框是可输入的...

综上所述，bug 就不好修了：

1. 不能保证按「回车」能弹 file dialog，IE8/9 只认空格
2. 不能保证一次「tab」能让它得到焦点
3. 不能保证用 JS 强行 focus 之后按「回车」和「空格」能弹出 file dialog（IE6/7）
4. 我日，我只能日啊！

浏览器不兼容，错却要我来背...

但还是坚持一下用一些 JS 来 walk around 看看吧...

bug 是说「tab」聚焦后，按「回车」无效。我的节点又是插进去到页面的节点的，所以一般只需要在页面节点的 `focus` 事件上把 `file input` 聚焦；然后使得它能响应「回车」就行了。

这样的话，只需要处理 IE，因为 IE 不响应回车，而 bug 又要求「回车」。这就好办了，只需要判断 `keyCode` 即可。至于「空格」，由于当焦点在按钮上也会弹，所以就不能再考虑了。但是出于安全的考虑，有些浏览器 很可能不让你通过代码触发 file dialog...

先试一下。

## 代码触发 file dialog

```
fileInput.click();
```

Firefox 很奇怪，有时会弹一个警告黄条，说弹出窗口被阻止，有时又完全没有反应；Chrome 和 Opera 没有反应；IE 和 Safari 都可以弹 file dialog。

实验成功，反正我们只需要对 IE 进行处理，由于 input 天生支持键盘事件，所以我们只需要绑定一个键盘事件，然后判断 `keyCode` 如果等于 `13` 的话，就手动触发 `click`。
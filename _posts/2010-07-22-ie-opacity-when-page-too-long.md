---
layout: post
title: IE 的透明度极限
date: 2010-07-22 15:07:32
categories: frontend
tags: develop
---

自从越俎代庖给 Artemis 写了几个通用的 JS 组件，可算是知道了什么叫吃力不讨好，好多 bug...有些确实是我的不好，但是就像之前说到的那个 IE 下 `remove` 之后取 `parent` 的问题...

今天还碰到了另外一个 IE 下的问题：Dialog 的 overlay 层，黑背景色 + 60% 的透明度，整个黑掉了！也就是说透明度没起作用。

问题发生在 Artemis 的 contact模块：

![](/images/posts/artemis_contact.png)

因为问题只发生在 Contact 模块，我第一时间想到的是 CSS 冲突。实在信不过另外几个模块的 CSS，可是找了很久都没找到冲突在哪里；然后还发现另外一个奇怪的现象，有的时候是好的，有的时候就是坏的，有这么个规律：

1. 切换到「All」以外的任何一个 Tab
2. 选择一个「Contact」点进去
3. 返回，就好了...这时候会停在点击的 Tab，不是「All」

会不会是「All」的时候节点太多了？于是想在我们的 Storage 模块先添加很多节点再弹 Dialog。

可是一个不小心写错了代码，导致不得不强行关闭 IE...后来经人点拨，会不会是 Contact 太多，页面太长导致的？Contact 里面的节点是 Lazy Load 的，但是会把高度留出来，就导致整个页面相当长 - 不是一般的长...

冲着这个思路，试验了一把 - 果然！哦～歌颂你! 万恶的IE...

我只是试了大概的一个数字 - 十万，黑了

```js
var div = document.createElement("div");
div.style.width = "100%";
div.style.height = "<span style="color:blue;"><strong>100000px</strong></span>";
div.style.backgroundColor = "#000";
div.style.filter = "alpha(opacity=60)";
div.style.position = "absolute";
div.style.top = 0;
div.style.left = 0;
document.body.appendChild(div);
```

通过几次实验发现在 6 万到 7 万之间，然后是 6.5 万到 7 万之间...发现有 655XX 的迹象！难道？试了一下，果然！

极值就是 2 的 16 次方 - 65536。65535 都还是好的...

以上是在 IE8 下测试的结果，IE6 也有个极值 - 2的 15 次方 - 32768，但又有点不一样，IE6 是小于等于 2 的 15 次方的时候有效，大于 2 的 15 次方的时候，Dialog 在 IE6 下也是全黑。

原来微软还在16位徘徊呢？不都...64 位了吗，Win8 还号称 128 呢...
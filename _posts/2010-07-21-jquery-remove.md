---
layout: post
title: jQuery 对象 remove 之后...
date: 2010-07-21 13:07:38
tags: develop
---

今天碰到一个 bug，在 IE 下才有。

是我的 bubble 插件，HF 的同事帮我找到了症结在哪里，一个节点通过 jQuery 的 remove 方法之后，取它的 parent，IE 下居然是存在的！

于是我做了以下实验：

## 1. Firefox

代码

```js
var fuck = $("#fuck").remove(),
    fuckP = fuck.parent();

console.info(fuckP);
for (var x in fuckP) {
    if (typeof fuckP[x] !== "function") {
        console.info(x, ":", fuckP[x]);
    }
}
```

效果

![](/images/posts/jquery_remove_parent_ff.png)

符合常规思维：parent 是空的 - 因为已经 remove 掉了嘛

## 1. IE8

一样的代码

```js
var fuck = $("#fuck").remove(),
    fuckP = fuck.parent();

console.info(fuckP);
for (var x in fuckP) {
    if (typeof fuckP[x] !== "function") {
        console.info(x, ":", fuckP[x]);
    }
}
```

效果

![](/images/posts/jquery_remove_parent_ie8.png)

醒目的 **length:1**

看看这个 parent 究竟是何方神圣：

```js
var fuckP_0 = fuckP.get(0);
for (var x in fuckP_0) {
    if (typeof fuckP[x] !== "function" && fuckP_0[x] !== null && fuckP_0[x] !== undefined) {
        console.info(x, ":", fuckP_0[x]);
    }
}
```

效果

![](/images/posts/jquery_remove_parent_ie8_parent.png)

出错了...不管它，已经够了 - **nodeType:11**。11 表示 `DOCUMENT_FRAGMENT_NODE`。你可以在 Firebug 里运行这段代码看 NodeType 的含义：

```js
for (var x in Node) {console.info(x, Node[x]);}
```

那么是 jQuery 的问题呢，还是万恶的 IE 的问题呢？进一步研究，完全抛弃 jQuery，代码如下：

```js
var fuck = document.getElementById("fuck");
fuck.parentNode.removeChild(fuck);
console.info(fuck.parentNode.nodeType);
```

效果

![](/images/posts/jquery_remove_parent_ie8_reason.png)

果不出所料，歌颂你，万恶的 IE。IE8 都这个操性，IE6 就不必测了，不清楚为什么 IE 要这么做。
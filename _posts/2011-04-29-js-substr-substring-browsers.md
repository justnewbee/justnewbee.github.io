---
layout: post
title: JS 中的小 substr 与大 substring 的区别
date: 2011-04-29 06:04:26
tags: JavaScript
---

碰到一个 bug，说在 IE 下面对文件进行「new version」的操作，即使选择了正确的文件类型，都会报文件类型不符。分析了一会儿，发现问题出在我用了 `substr` 的缘故。

为什么要用小 `substr` 呢？很简单，因为我判断用的是后缀名，在后缀名长度一定的情况下，使用 `substr` 很方便，比如 `fuck.jpg` 只需要 `substr(-3)` 就是 `jpg` 了。

可是，我错了...不，是 IE 错了。

其实我本来几乎不用 `substr` 的，只是偶然间看到了它在参数为负的情况下的特殊能力。

碰到这个 bug，于是就想把 `substr` 和 `substring` 两者之间的区别弄弄清楚。

首先在 [devguru](http://devguru.com) 上查了它们之间的区别

> **substr**
> Syntax: object.substr(start[, length])
> This method extracts the characters from a string beginning at the specified start index for the specified number of characters.
>
> **substring**
> Syntax: object.substring(indexA, indexB)
> This method returns the characters in a string between two specified indices as a substring.

其实他这里的解释也不完全正确
比如 `substring` 的第二个参数其实也是可选的，如果第二个参数省略，会认为它大于等于字符串的长度。

为了进一步了解它们在不同浏览器上的行为，我写了以下代码：

```js
var str = "123456789";
// one param - 0 +
console.info("(-3): "" + str.substr(-3) + "" - "" + str.substring(-3) + """);
console.info("(0): "" + str.substr(0) + "" - "" + str.substring(0) + """);
console.info("(3): "" + str.substr(3) + "" - "" + str.substring(3) + """);
// two prams both -, &lt; = &gt;
console.info("(-6, -3): "" + str.substr(-6, -3) + "" - "" + str.substring(-6, -3) + """);
console.info("(-3, -3): "" + str.substr(-3, -3) + "" - "" + str.substring(-3, -3) + """);
console.info("(-3, -6): "" + str.substr(-3, -6) + "" - "" + str.substring(-3, -6) + """);
// two params both +, &lt; = &gt;
console.info("(3, 6): "" + str.substr(3, 6) + "" - "" + str.substring(3, 6) + """);
console.info("(3, 3): "" + str.substr(3, 3) + "" - "" + str.substring(3, 3) + """);
console.info("(6, 3): "" + str.substr(6, 3) + "" - "" + str.substring(6, 3) + """);
// two params + -
console.info("(-3, 6): "" + str.substr(-3, 6) + "" - "" + str.substring(-3, 6) + """);
console.info("(3, -6): "" + str.substr(3, -6) + "" - "" + str.substring(3, -6) + """);
```

并分别在 IE、Firefox、Chrome、Safari 、Opera 上做了测试，运行效果如图：

![](/images/posts/substr_substring_browsers.png)

结果很明显了，IE 就是那么与众不同，问题都出在第一个参数为负，第二个参数为正（空也可以看作为正）的情况。其他的浏览器都会从后往前数，只有 IE 把它当成 `0` 看。

所以为了避免 bug，我们还是用 substring 比较保险，至少它没有浏览器兼容问题。

诅咒 IE！
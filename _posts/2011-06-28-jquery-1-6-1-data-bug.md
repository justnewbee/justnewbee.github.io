---
layout: post
title: jQuery 1.6.1 data 方法的 bug
date: 2011-06-28 17:06:44
tags: develop, javascript, jquery
---

我有个原则，就是除了代码中变量名之外，轻易不会用骆驼命名。所以，很多字符串常量值我一般会全部小写，中划线或下划线链接。而中划线是我的最爱，因为它不会被用于变量名，可以放心地做全局替换。

jQuery 的 `data()` 是个很有用的方法，对应的有个 `removeData()` 的方法。

今天发现了它的一个 bug，当 data key 中有中划线的时候，设 data 的时候没问题，取的时候，以及移除的时候都不行。

我做了如下试验：

```js
node.data("x-y", "x-y").data("x.y", "x.y").data("x_y", "x_y").data();
```

结果 `x-y` 就给硬生生转成了 `xY`...调用 `.data("x-y")` 能正确返回，但 `.removeData("x-y")` 却不能成功。

jquery 版本是 1.6.1，然后试了 `1.4.2`，并没有问题。

在 jQuery 上提了 [bug ticket](http://bugs.jquery.com/ticket/9691)。
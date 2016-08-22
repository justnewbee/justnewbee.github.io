---
layout: post
title: 什么是 replaced / non-replaced element
date: 2014-12-09 10:49:51
categories: html
---

# line-height 引出的问题

不得不承认，虽然使用了很久的 HTML 和 CSS，但还有很多看似很基础的概念了解得不够透彻，比如 CSS 的 `line-height` 属性。但本文讲的不是 `line-height`，而是由它引出的另一个东西 - replaced / non-replaced element。

因为在查阅 [MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/line-height) 的时候，发现了下面的话：

> On block level elements, the line-height CSS property specifies the minimal height of line boxes within the element.
>
> On non-replaced inline elements, line-height specifies the height that is used in the calculation of the line box height.
>
> On replaced inline elements, like buttons or other input element, line-height has no effect.

`block`、`inline`、`inline-block` 没问题，但 `non-replaced` 和 `replaced` ...

# replaced element

## MDN 上的定义

就近查阅 MDN 正好有 [关于replaced element的文章](https://developer.mozilla.org/en-US/docs/Web/CSS/Replaced_element)，文章很短：

> In CSS, a replaced element is an element whose representation is outside the scope of CSS. These are kind of external objects whose representation is independent of the CSS. Typical replaced elements are `<img>`, `<object>`, `<video>` or form elements like `<textarea>` and `<input>`. Some elements, like `<audio>` or `<canvas>` are replaced elements only in specific cases. Objects inserted using the CSS content properties are anonymous replaced elements.
>
> CSS handles replaced elements specifically in some cases, like when calculating margins and some auto values.
>
> Note that some replaced elements, but not all, have intrinsic dimensions or a defined baseline, which is used by some CSS properties like vertical-align.

## W3C 的定义

[W3C的文章](http://www.w3.org/TR/CSS2/conform.html#replaced-element) 中关于 replaced element 的这一段有些晦涩难懂...

> An element whose content is outside the scope of the CSS formatting model, such as an image, embedded document, or applet. For example, the content of the HTML IMG element is often replaced by the image that its "src" attribute designates. Replaced elements often have intrinsic dimensions: an intrinsic width, an intrinsic height, and an intrinsic ratio. For example, a bitmap image has an intrinsic width and an intrinsic height specified in absolute units (from which the intrinsic ratio can obviously be determined). On the other hand, other documents may not have any intrinsic dimensions (for example, a blank HTML document).

> User agents may consider a replaced element to not have any intrinsic dimensions if it is believed that those dimensions could leak sensitive information to a third party. For example, if an HTML document changed intrinsic size depending on the user's bank balance, then the UA might want to act as if that resource had no intrinsic dimensions.

> The content of replaced elements is not considered in the CSS rendering model.

## 反过来看看 non-replaced element

StackOverflow上 [关于non-replaced element的讨论](http://stackoverflow.com/questions/12468176/what-is-a-non-replaced-inline-element)，有更好的参考价值。「it is best to read "replaced" as "embedding"」这句话，很好地阐述了什么是 replaced element：它的内容是嵌入进来的。

# 总结

关于 `replaced element`，概括来说有以下几点：

1. 多为嵌入外部的对象，典型的有：`<img>`、`<object>`（那就肯定有 `<embed>`）、`<audio>`、`<video>`、`<canvas>`，像 `<textarea>`、`<input>`、`<select>` 这样的form元素（跟历史有关，早期的浏览器使用的系统的东西来实现它们，导致无法被 CSS 监管到）；
2. 很多属性不受 CSS 控制；
3. 使用 CSS 的 `content` 属性插入的对象是一个 replaced element（`:before`、`:after`）；
4. CSS 对 replaced element 的一些计算会特殊处理，如 `margin` 和一些 `auto` 值；
5. 有些（不是所有）的 replaced element 会有自己固有的尺寸，或用于 `vertical-align` 等属性的 baseline
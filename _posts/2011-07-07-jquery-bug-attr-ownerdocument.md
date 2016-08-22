---
layout: post
title: jQuery 1.6 的版本间兼容性 bug .attr("ownerDocument")
date: 2011-07-07 04:07:18
tags: develop, javascript, jquery
---

今天又发现一个 jQuery 的 bug，1.6.2 `.attr("ownerDocument")` 每次都返回 `undefined`，1.4.2 是好的，提了 [ticket](http://bugs.jquery.com/ticket/9764)。

其实这是 jQuery 在升级的过程中，对 `prop()` 和 `attr()` 进行了更合理的规范造成的，不能算是 bug。
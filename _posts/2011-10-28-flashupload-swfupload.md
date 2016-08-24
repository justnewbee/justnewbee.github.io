---
layout: post
title: FlashUpload 对于 SWFUpload 的改进
date: 2011-10-28 08:10:01
categories: frontend
tags: as, js, upload
---

因为 [彻底改写 SWFUpload](/2011/10/28/totaly-modify-swfupload.html)，这里稍作说明和总结。

`FlashUpload` 是在 `SWFUpload`（2.2.0 2009-03-25）的基础上改的，这个版本的 license 还是 MIT 的，所以没有限制。

这个版本的 SWFUpload 有几个比较著名的 bug：

1. 无休止的JS报错「object is required」、「null is not an object...」之类。主要发生在 IE 下把 flash 或其所在的节点移除的时候，大概是因为虽然 flash 的节点被删掉了，但是内存中它依然存在，而 AS 中有个循环的 TestExternalInterface 的 JS 调用会取 flash element；非 IE 浏览器没有这个问题 - 解决的办法是新加 AS `Destroy` 接口，它会停止所有的 timer，在移除节点之前通过 JS 调用
2. IE 下经常发生「ExternalInterface Reinitialized」。还是 IE 下的内存管理的问题 - 解决的办法是每次用新的 flash，prevent cache 永远得是 `true`
3. 非 IE 的情况下，点击弹出文件浏览窗口并将其关闭后，浏览器失焦 - 完全没有办法...IE 因为弹出来的是 modal dialog，反而幸免于难
4. tab 陷阱。Firefox / Chrome...两种情况，1) 自然「tab」，从别处「tab」进 upload button 之后，无法「tab」出去；2) 上述第三条之后「tab」 - 解决的办法是，新加 JS `tabbing:Function` 参数，AS 内部绑定 stage 的 keydown 事件，keycode 为 `9` 的时候，调用 JS 的 `tabbing`。对于 1)，这样的解决方案可行，对于 2)... 必须等用户把点击浏览器让浏览器重新获得系统的焦点，才能发现已经「tab」出来了
5. 在初始化完成之前调用 Flash 的方法导致 JS 错误。JS 创建 SWFUpload 的实例到该实例初始化完成（即加载完成）是需要时间的，但时候有时候，由于业务的需要，需要在实例化之后对它进行一些操作或设置，如果调用的方法最终调用了的 Flash 内部的方法，则会报 JS 错误 - 解决的方案是在初始化完成之后，所有的 `CallFlash` 的操作都缓存起来，ready 了之后再逐个执行
6. IE 下的 title bug，如果 url 中带有 `#`，IE 下的 `document.title` 就会很神奇的被改掉，这个变化一般发生在加载完成和鼠标 `mousedown` 的时候 - 解决方案，在那两个时刻，调用 JS 端的新接口 `ieHashTitle`，当然，你得知道当前的 title 应该是什么，并且只需为 IE 做这个事情。
7. 文件大小 `NaN`，报错为 `0size`，当文件大小超过 `4G` 的时候就会有这个问题 - 解决的方案就是新给个错误代码

以上的 bug 除了 3 在 `FlashUpload` 中都有改。

除此之外，`FlashUpload` 把 `SWFUpload.js` 中的内部 `Console` 去掉了，而改用了 `window.console`，当然 `window.console` 如果不存在，就干脆不 log 了。

`SWFUpload` 的 `debug` 和 `debugEnabled` 着实叫人混淆不堪，干脆把 `debugEnabled` 去掉，把用于打 `log` 的方法 `debug()` 改成就叫 `log`，而把设置中的 `debug` 作为 `boolean` 参数。

## 总结一下

* flash 文件大小 19K -> 14K
* AS 文件个数 3 -> 2
* AS 代码总行数 121 + 112 + 1538 = 1771 --> 113 + 982 = 1095 少了 676 行，且后者包括新添加的那许多行注释
* JS 代码行数 994 --> 791 少了 203 行，同样，后者包括新添加的那许多行注释

当然，这还不是最终结果，可是已经能够能看到一些区别了。
---
layout: post
title: IE 下 AS ExternalInterface 对 id 的变态要求
date: 2011-11-22 07:11:39
categories: bug
tags: as, js, upload
---

`FlashUpload` 改完也好久了，这几天都在着手上层 UI 及业务逻辑的封装。

今天突然想去 IE9 上测一下，打开测试页面，傻眼了...不断循环的一堆 JS 错误，大致如下：

> SCRIPT5007: Unable to get value of the property 'SetReturnValue': object is null or undefined
> SCRIPT5007: Unable to get value of the property 'SetReturnValue': object is null or undefined
> SCRIPT5007: Unable to get value of the property 'SetReturnValue': object is null or undefined
> ...
> SCRIPT5007: Unable to set value of the property 'Destroy': object is null or undefined
> SCRIPT5007: Unable to set value of the property 'TestExternalInterface': object is null or undefined
> ...

联想到最近的可能影响到 IE 上面的改动只有一个，那就是把 `window[this._movieName] = movieElement;` 这行去掉了，加回之后，仍旧报错。

Google 之，文章挺多的还，最靠谱的应该在这里：<http://stackoverflow.com/questions/7523509/script5007-unable-to-get-value-of-the-property-setreturnvalue-object-is-null>

> Make sure you specify the id tag (it must have the same value as the name tag). Although Adobe writes that id is an optional tag, Internet Explorer needs the id to address the swf object with the javascript-flash interface.

这句话提醒了我，因为，为了使用 HTML 的 UI 特性，把原先的只有 flash 的方式改掉了，用一个 span 包住 flash 并且把 id 给了 span 改了取 flash movie 的接口。而 Flash 调用 JS 的接口也改成了静态方法，并且由 JS 告诉 AS 你该调哪个方法。所有其他的浏览器都 OK，就 IE 不行，只能把 id 加回给 flash 的 object，同时改了相应的接口。再测试 好了...
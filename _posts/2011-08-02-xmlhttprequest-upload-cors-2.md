---
layout: post
title: XMLHttpRequest Upload及CORS研究 (2)
date: 2011-08-02 16:08:19
categories: 倒腾
tags: ajaxupload, javascript
---

前一篇写得过长了... 实在对不住懒人这个称号。

上文说到对于 AJAX 上传的三种方式本身及它们在 CORS 的情况下的表现没有深入的研究，这里继续。

三种方式：

1. `Content-Type` 为 `application/octet-stream` 发送且只能发送 `File` 实例
2. 使用 `FormData` 发送 `File` 实例对象及其它数据
3. 使用 `FileReader` 本地读取文件数据，并手动拼装文件数据和其他数据

其中，方法 2 和 3 实质上没有区别，都是把数据模拟成 `Form` 提交时候的样子，而 3 则是模拟 2。

方式 1 比较死板，其他的数据似乎只能放到 `request` 的 `header` 里，经过测试，不能适应 CORS（至少我没有成功）。在 Chrome 下可以得到错误 - `Request header field X-File-Name is not allowed by Access-Control-Allow-Headers.`。

方法 3 看来很美，但出了问题就没办法解决。我碰到过两个问题，都出现在文件上传开始之前，某个 Excel 文档 - `String contains an invalid character" code: "5`；某个 EXE 安装包 - `File Read Error`。且 `FileReader` 在读取大文件的时候有延时。

再来看比较值得期待的方法 2 吧。在 [这篇文章](http://tiffanybbrown.com/xhr2/) 里看 `FormData` 的支持率：

![](/images/posts/xhr2_browser_support.png)

IE 毋庸置疑的不支持；Opera 可惜了，这么好的浏览器...

经过测试，方法 2 对于 CORS 的支持是不错的。但是，我上传一个比较大的文件，经过一段时间的毫无反应之后，发现后台报错 `java.net.SocketTimeoutException: Read timed out`。

难道真的无解？还是，我们继续等浏览器来解决这些 bug 吧...
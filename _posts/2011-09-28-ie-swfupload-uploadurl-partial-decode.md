---
layout: post
title: swfupload 上传 url 仅部分 decode
date: 2011-09-28 11:09:22
categories: frontend
tags: as, ie, upload
---

swfupload 的 AS 源码已经被我改得面目全非了，我也从原来的免费的 flashdevelop 编译转向了 adobe flashbuilder 的编译。没想到正是这个转变导致了一个 bug。

首先我们的 `uploadUrl` 对于每个文件都是不同的，是由一个固定的前缀加上 `encode` 过的文件名作为 url，比如 _王建春(2).docx_ 的 `uploadUrl` 可能是 `http(s)://server_url/%E7%8E%8B%E5%BB%BA%E6%98%A5%282%29.docx?params...`。

可是独独在 IE 下不能上传，报错中含有信息 `...http(s)://server_url/王建春%282%29.docx...`，如果说它 decode 过了吧，也只是部分 decode，只把中文给 decode 了。

源码看过了，没有可疑的地方。目前的版本跟之前的版本，除了代码有很大的精简外，唯一的不同就是编译器...

试验了之后，发现果然是...我只能说，这个 bug 修得很搞笑。
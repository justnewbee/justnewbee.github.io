---
layout: post
title: xampp apache 的一堆 /xampp/apache
date: 2012-12-04 13:12:43
categories: 倒腾
tags: develop, software
---

项目需要，用了一下 apache http server，偷懒搞了个 xampp，心想以后可能还要用它里面的很多东西，放一起也挺好。

下了个 zip 包，解压到 `d:/develop/xampp`，起 apache 老是出错。

首先是 `ServerRoot` 必须设成绝对路径 `d:/develop/xampp/apache`；然后是 `DocumentRoot` 也须绝对 `d:/develop/xampp/htdocs`；再然后还要把 `htdocs` 下面的 `index.php` 改成 `index.php_` 大概是因为我的 php 没启动？

可是运行的时候还是会报如下错误：

> Warning: DocumentRoot [D:/xampp/htdocs] does not exist

其实到了这里就已经很明显了，跟很多同志写代码一样，入口不唯一，不知道这是 apache 的问题还是 xampp 的问题。

在 `apache` 目录下搜 `/xampp/htdocs`，果然还有好几处，改之，则问题消失。
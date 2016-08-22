---
layout: post
title: 命令行获得当前的路径
date: 2012-09-12 05:09:26
tags: tech
---

虽然不是经常，但偶尔还是需要写批处理文件，获得当前的目录有些时候十分有用。

Windows 在 CMD 中使用 `%cd%`，大小写无关；批处理文件中使用 `%~dp0` 可获得该批处理文件所在的目录。

Apple 和 Linux 的 terminal 中使用 `` `pwd` ``。Apple 大小写无关，Linux 似乎必须小写。

为了避免出错，尽量使用小写的吧。
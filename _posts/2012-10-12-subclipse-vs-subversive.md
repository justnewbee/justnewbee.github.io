---
layout: post
title: subclipse vs subversive
date: 2012-10-12 06:10:02
tags:  develop, eclipse, software
---

公司最近把项目迁移到 svn 上，老要慢一步（而且慢好久...）。svn 都出来好久了，估计再过几年又会想到用改 git。eclipse 都宣布即将用 git 替代默认的 cvs 了。算了，不提...

eclipse 的 svn 插件，用的最多的应该是 [subclipse](http://subclipse.tigris.org) 和 [subversive](http://www.eclipse.org/subversive/) 这两个了。似乎 subclipse 的资历更老一点。至于为什么让 subversive 混成 eclipse 官方的东西就不得而知了。

我两个都试用了一下，发现还是 subclipse 比较符合我的风格。

## 安装

很明显 subclipse 比 subversive 方便。subclipse 是一次性搞定；而 subversive 除了安装 subversive 之外，还需要安装 connecter，不过两个都提供了离线安装 zip 还是挺为我这样的人考虑的。

## 插件大小

subclipse zip 为 16M，subversive zip 19M + connector zip 20M = 39M > subclipse * 2。

但 subclipse 的功能一点都不逊色，某些方面反而更出色。

## checkout

用惯了 eclipse 的 cvs 的肯定会喜欢 subclipse，因为它的 checkout 流程跟 cvs 几乎一模一样（看起来似乎 subclipse 更官方一点啊）。

## `.svn`

[乌龟 svn](http://tortoisesvn.net) 自版本 1.7 之后，checkout 的文件只有在根目录下有 `.svn` 文件夹了，而不像之前的版本一样每层目录都有一个。subclise 是 1.7 以后的乌龟，而 subversive 则是老乌龟。所以要想配合新乌龟就用 subclipse。

嗯，我挺 subclipse。
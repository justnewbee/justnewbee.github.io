---
layout: post
title:  "Learning vim"
date:   2014-11-04 22:12:51
categories: jekyll update
---

一直停留在[vim](http://www.vim.org/)的浅层 （只要是在vim里面用上下左右箭头 而不是kjhl的 都得低头承认啊） 最近想好好用用它 毕竟人家是“编辑器之神”

# 用过的编辑器

## Windows + Mac

* [Sublime](http://www.sublimetext.com) - 相信是目前最火的编辑器了 创新多点编辑和minimap 可以说引领了编辑器的革命 已经形成自己的生态圈 不花钱可以一直用 偶尔会弹个框让你注册 插件多
* [Brackets](http://brackets.io) - 最近蛮喜欢的编辑器 饿肚皮公司生产 界面喜欢 特点是慢 唉 插件还挺多的 只是装插件经常出错
* [Atom](https://atom.io) - Github出品 让很多人尤其是前段开发激动得一塌糊涂 特点是新 bug多 功能不全 插件不多
* [UltrEdit/UltraStudio](http://www.ultraedit.com/) - 早期的王者 最近似乎有挺大的改进 出Mac版了 但没用过
* [LightTable](http://lighttable.com/) - 也是一个新生力量 界面清爽 但没有配置项 这点不爽

## Windows

* [HippoEdit](http://www.hippoedit.com) - 号称是Windows上的Best 个人喜欢的#2 退出时可以自动备份数据 跟Sublime一样 也是可以不花钱一直用 有丰富的插件
* [Nodepadd++](http://notepad-plus-plus.org) - 免费且功能强大
* [Editplus](http://www.editplus.com) - 一直觉得UI很丑 据说是php利器 $35 估计大部分人用的是注册机
* [EditPad Pro](http://www.editpadpro.com) - JGSoft出品 用过一段子时间 蛮有特色 还不错 $49.95 有不要钱的Lite版
* [PSPad](http://www.pspad.com/) - 挺好的一个免费编辑器
* [AkelPad](http://akelpad.sourceforge.net/en/index.php) - 因为用totalcmd而认识的一个编辑器 小巧快速 网站似乎被某匪墙了 天理呢

## Mac

* [TextMate](http://macromates.com) - 装了但不怎么用
* [BBEdit](http://www.barebones.com/products/bbedit/index.html) - 一样装了但不怎么用 要$ TextWrangler似乎是它的轻量免费版
* [EMacs](http://www.gnu.org/software/emacs) - 神之编辑器 也就是神用的 我只是凡人一条
* [Textatic](http://www.textasticapp.com) - 有兴趣可以研究一下

这里不是来比较各个浏览器哪个好哪个不好；我认为只要是编辑器，那就是好的，毕竟不是每个人都能写出编辑起来的。每个人按自己的喜好，选择几个适合自己的就行了。我选择编辑器的标准是：

1. 最好不要钱，或者说不要逼我掏钱（不给钱就不给用的我不喜欢）
2. 图标要能第一时间抓住眼球
3. 要有丰富的可配置项，以下是必须支持的
    * 文件编码
    * 空白字符可见 而且要好看
    * TAB size
    * 可以配置TAB不要替换成空格（最好默认采用不替换TAB模式）
    * 支持语法高亮 最好能自定义某些高亮规则
    * 快捷键自定义
4. 最好具有以下功能
    * 丰富的插件
    * 自动补全
    * Markdown预览
    * 正则查找与替换

# vim

## 系统自带的编辑器

好吧，Mac和Linux其实很流氓地给你装了一些编辑器，甚至你都不知道有这么回事，vim就是其中之一，当你在terminal中输入`vi`或`vim`的时候，就会调出系统自带的vim编辑器。

其实还有很多，你一可以逐一试一下：`emacs`、`nano`、`pico`。Linux下似乎有`gedit`。

## 自带教程

在terminal里输入命令`vimtutor`会调出vim自带的入门教程，它会带你“play around”。

## 不错的资料

* [vim学习资料的搜集](http://www.openvim.com/index.html)
* [入门](http://tips.webdesign10.com/another-vim-tutorial)
* [蛮全的一个学习列表](http://alvinalexander.com/linux/vi-vim-editor-tutorials-collection)

# movement (未完)

https://bitbucket.org/tednaleid/vim-shortcut-wallpaper/raw/tip/vim-shortcuts.png

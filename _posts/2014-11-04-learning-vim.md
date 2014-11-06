---
layout: post
title:  "学习vim #0 - 引子"
date:   2014-11-04 22:12:51
categories: jekyll update
---

一直停留在[vim](http://www.vim.org)的浅层（只要是在vim里面用上下左右箭头，而不是`kjhl`的，都得低头承认啊），最近想好好再搞一下，毕竟人家是“编辑器之神”啊。

# 用过的编辑器

用过不少的编辑器，这里做一下记录。

## Windows + Mac

* [Sublime](http://www.sublimetext.com) - 相信是目前最火的编辑器了，创新多点编辑和minimap，可以说引领了编辑器的革命；插件多，已经形成自己的生态圈；不花钱可以一直用，偶尔会弹个框让你注册
* [Brackets](http://brackets.io) - 最近蛮喜欢的编辑器，饿肚皮公司生产，界面很清新；特点是慢，唉；插件还挺多的，只是装插件经常出错
* [Atom](https://atom.io) - Github出品，让很多人尤其是前段开发激动得一塌糊涂；特点是新，bug多，功能不全，插件还处于爆发期
* [UltrEdit/UltraStudio](http://www.ultraedit.com) - 早期的王者；最近似乎有挺大的改进，出Mac版了，但没用过
* [LightTable](http://lighttable.com) - 也是一个新生力量，界面清爽；但似乎没有配置项，这点不爽

## Windows

* [HippoEdit](http://www.hippoedit.com) - 号称是Windows上的Best，个人喜欢的#2，退出时可以自动缓存数据，不会干扰你退出；有丰富的插件；也是可以不花钱一直用，只是启动的时候会有个提醒
* [Nodepadd++](http://notepad-plus-plus.org) - 免费且功能强大
* [Editplus](http://www.editplus.com) - 一直觉得UI很丑；据说是php利器；35美大洋，估计大部分人用的是注册机
* [EditPad Pro](http://www.editpadpro.com) - JGSoft出品，用过一段子时间，蛮有特色，还不错；49.95美大洋，有不要钱的Lite版
* [PSPad](http://www.pspad.com/) - 挺好的一个免费编辑器
* [AkelPad](http://akelpad.sourceforge.net/en/index.php) - 因为用totalcmd而结识的编辑器，小巧快速，更新频度高；网站似乎被某匪墙了，天理不在

## Mac

* [TextMate](http://macromates.com) - 装了但不怎么用
* [BBEdit](http://www.barebones.com/products/bbedit/index.html) - 一样装了但不怎么用，要大洋，TextWrangler似乎是它的轻量免费版
* [EMacs](http://www.gnu.org/software/emacs) - 神之编辑器，也就是神用的，我只是凡人一条
* [Textatic](http://www.textasticapp.com) - 有兴趣可以研究一下

这里不是来比较各个浏览器哪个好哪个不好；我认为只要是编辑器，那就是好的，毕竟不是每个人都能写出编辑起来的。每个人按自己的喜好，选择几个适合自己的就行了。我选择编辑器的标准是：

1. 最好不要钱，或者说不要逼人掏钱（不给钱就不给用的有些流氓）
2. 图标和界面要能第一时间抓住眼球
3. 要有丰富的可配置项，以下是必须支持的
	* 文件编码
	* 空白字符可见，而且要好看
	* TAB size
	* 可以配置TAB不要替换成空格（最好默认采用不替换TAB模式）
	* 支持语法高亮，最好能自定义某些高亮规则
	* 快捷键自定义
4. 丰富的插件
5. 具有以下功能（内置或通过插件提供）
	* emmet编程方式
	* 自动补全
	* Markdown预览
	* 正则查找与替换
	* 记住上次打开时的状态
	* 自动缓存，即使不小心关掉也不会丢失数据；不要再退出的时候提醒说“怎么就要走了呢，保存一下呗”
6. 速度快，启动可以不要秒开，但编辑的过程中不能卡

# vim

虽然由于其使用门槛的问题导致vim的用户不如其他的很多编辑器，但如果真的用惯了，你可以在没有鼠标的情况下，极大地提高编辑效率。可以说，vim在键盘的操作性上几乎做到了极致。这其实也是我为什么要再好好学习一下vim的原因。

## 系统自带的编辑器

好吧，Mac和Linux其实很流氓地给你装了一些编辑器，甚至你都不知道有这么回事，vim就是其中之一，当你在terminal中输入`vi`或`vim`的时候，就会调出系统自带的vim编辑器。

其实还有很多，你一可以逐一试一下：`emacs`、`nano`、`pico`；Linux下还有`gedit`。

## 自带教程

在terminal里输入命令`vimtutor`会调出vim自带的入门教程，它会带你“play around”。

## 不错的资料

* <http://www.openvim.com/index.html> 里面的tutorial是非常棒的交互式学习教程
* <http://tips.webdesign10.com/another-vim-tutorial> 入门教程
* <http://alvinalexander.com/linux/vi-vim-editor-tutorials-collection> 蛮全的一个学习列表
* <http://derekwyatt.org/vim/tutorials> 视频教程 一个带口音的外国人搞的 但看视频需要翻墙
* <http://jrmiii.com/attachments/Vim.pdf> 一个比较全的vim思维导向图形式的cheatsheet
* <http://blog.interlinked.org/tutorials/vim_tutorial.html> 文字比较多的一个教程
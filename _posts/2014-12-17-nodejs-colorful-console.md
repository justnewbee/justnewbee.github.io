---
layout: post
title: nodejs 打印带颜色的 console 信息
date: 2014-12-17 22:59:31
categories: nodejs console
---

一个简单的功能，更多情况下，我更喜欢用原生的东西，而不是急着找第三方的支持。

在 `grunt` 脚本中，如果要打印出带颜色的 log，我更倾向于使用如下的方式：

```js
console.log("the \x1b[43mParty\x1b[0m is fucking \x1b[31m%s\x1b[0m u know?", "shitty");
```

可以在 `node` 的 `REPL`（Read-Eval-Print-Loop，`node` 的可交互运行环境）里执行以上的代码看效果（如果有人跟曾经的我一样是个 `node` 白痴，那就让我这个白痴减1来「指导」一下：首先确认 `node` 已安装，然后在你的命令行工具里输入 `node`，回车，这样就进入了 `REPL` 了，可以使用命令 `.help` 来看看怎么玩）。

这样的代码虽然看起来很不友好，不容易读懂，但对于简单的任务来说，已经足够。复杂的任务可以使用第三方的包如 [npm colors package](https://npmjs.org/package/colors)。

上面的`\x1b[0m`表示「清除之前所有的样式」，不管前面叠加了几个样式，统统都不见了，如下：

```js
console.log("the \x1b[42m\x1b[34m\x1b[46mGreat Fire Wall\x1b[0m is most ugly and stupid!");
```

以下是可以使用的文本命令的参考：

前景色（文字颜色）：

```
\x1b[30m = 黑色
\x1b[31m = 红色
\x1b[32m = 绿色
\x1b[33m = 黄色
\x1b[34m = 蓝色
\x1b[35m = 洋红色
\x1b[36m = 青色
\x1b[37m = 白色
```

背景色：

```
\x1b[40m = 黑色
\x1b[41m = 红色
\x1b[42m = 绿色
\x1b[43m = 黄色
\x1b[44m = 蓝色
\x1b[45m = 洋红色
\x1b[46m = 青色
\x1b[47m = 白色
```

其他：

```
\x1b[0m = 清除样式
\x1b[1m = 加粗
\x1b[2m = 半透明
\x1b[4m = 下划线
\x1b[5m = 闪动
\x1b[7m = 取反：背景色变前景色 前景色变背景色
\x1b[8m = 看不见 但位置还留着
```

不难看出有以下规律可循：

1. 所有的文本命令都是以 `\x1b[` 打头
2. 以`m` 结尾
3. 中间一到两位数字
	- 两位数字是颜色 `4#` 表示背景色 `3#` 表示前景色
	- 前景 / 背景色的第二个数对应的颜色一样
	- 一位数字是控制

...其实我也只是在 [这里](https://coderwall.com/p/yphywg/printing-colorful-text-in-terminal-when-run-node-js-script) get 了的这个技能。
---
layout: post
title: 循环变非循环 - for/while/do_while
date: 2013-04-07 07:04:27
categories: develop
tags: algorithm
---

## 背景

### 循环组成

对循环的转化，不能只对循环语句进行转化，还必须考虑循环前后的语句。所以转化的是含有循环的代码块，可以划分为以下几个部分：

* `_preloop_` - 循环前
* `_loop_` - 循环本身
	- `_init_` - 循环初始化
	- `_cond_` - 循环判断
	- `_body_` - 循环体
	- `_adj_` - 循环变量修正
	- break - 提前结束循环
	- continue - 提前结束本次循环而开始下一次循环
* `_postloop_` - 循环后

以上用 `_xxx_` 作为标识，表示它可以是一段代码块，有时候，为了减小转化后代码的冗余度，或者为了提高代码的可读性，可以将代码块装入一个方法封装起来。

### 递归

循环转非循环的关键是递归，这一点是明显的。

定义一个递归的方法，它会进行循环条件判断（`_cond_`），在条件成立的情况下执行原循环体所做的事情（`_body_`)，并递归调用自身；条件不成立则不会调用自身，因而循环结束。

递归方法的定义不会引起循环的开始，需要调用一下该方法以启动循环。

### 应用场景

循环转非循环，由于递归的引入，除了增加代码量及弱化程序的可读性之外，没有别的用处。那为何还需要研究它呢？

假设有一个循环，需要输出每趟的状态。但不能一下子输出，而是慢镜头播放，即每隔一定的时间输出。不能控制 CPU 的运行速度，只能通过代码，有两种方式。

第一种，需要一个状态队列。循环仍旧按原来的进行，但每趟循环完成的时候需要将当时的状态保存在状态队列里。等循环完成后，每隔一段时间对相应的状态进行处理，可以用 `setTimeout` 或者 `setInterval`。这种方式的优点是简单，但需要额外的空间。根据问题的规模和复杂度，这个额外的空间，很有可能达到相当惊人的地步。

第二种就是这里讨论的循环转非循环的方式，使用 `setTimeout`（而且只能是 `setTimeout`)控制递归。这种方式的优点是不需要太多的额外空间，但可能很难读懂，有时候也很难写。

所以，循环转非循环的应用场景就在于此 - 慢镜头播放循环。

## 转化公式

先来看看最常见的三种循环的转化公式。

### 基础循环 - for

之前：

```js
_preloop_;
for (_init_; _cond_; _adj_) {
    _body_;
}
_postloop_;
```

之后：

```js
function loopFor() {
    if (_cond_) {
        _body_;
        _adj_;
        loopFor();
    }
}
_preloop_;
_init_;
loopFor();
_postloop_;
```

### 基础循环 - while

之前：

```js
_preloop_;
while (_cond_) {
    _body_;
}
_postloop_;
```

之后：

```js
function loopWhile() {
    if (_cond_) {
        _body_;
        loopWhile();
    }
}
_preloop_;
loopWhile();
_postloop_;
```

### 基础循环 - do_while

之前：

```js
_preloop_;
do {
    _body_;
} while (_con_);
_postloop_;
```

之后：

```js
function loopDoWhile() {
    if (_cond_) {
        _body_;
        loopDoWhile();
    }
}
_preloop_;
_body_;
loopDoWhile();
_postloop_;
```

### for 与 while

其实 `for` 循环实质上跟 `while` 循环是一样的，可以说 `for` 是 `while` 的一种特殊形式，可以如下进行转化：

```js
_preloop_;
_init_;
for (;_cond_;) { // 相当于while
    _body_;
    _adj_;
}
_postloop_;
```

把 `_init_` 看作 `_preloop_` 的一部分，`_adj_` 为 `_body_` 的一部分，以上 `for` 循环就是一个标准的 `while` 循环。

### 纯粹转化公式

没有任何干扰（`break`，`continue` 或 `setTimeout`）的条件下进行的转化很纯粹，转化后的代码与转化前的代码效果完全相同。比较一下三个转化公式，不难发现它们很像：

由于 `_postloop_` 必须在 `loopXxx` 方法执行完毕后才运行，而 `loopXxx` 方法执行完毕的条件是它内部的那一个隐藏的 `else`，为了能在慢镜头播放的情况下也是同样的顺序（`setTimeout` 会让后方的代码先运行)，有必要把 `_postloop_` 往前提一下。

把 `for` 看成一个特殊的 `while`，可以归纳成如下对三种循环都适用的公式：

```js
function loop() {
    if (_cond_) {
        _body_;
//      _adj_; // for
        loop();
    } else {
        _postloop_;
    }
}
_preloop_;
//_init_; // for
//_body_; // do...while
loop();
```
### setTimeout

`setTimeout` 的情况与之前的纯粹转化公式只有一个区别，就是如何递归。

```js
function loop() {
    if (_cond_) {
        _body_;
//      _adj_; // for
        setTimeout(loop, ms);
    } else {
        _postloop_;
    }
}
_preloop_;
//_init_; // for
//_body_; // do...while
loop();
```

事实上，JavaScript 有它很特殊的特性，可以定义这样一个局部的 `setTimeout` 方法，使得跟没有 `setTimeout` 一样。

```js
function setTimeout(fn, ms) {
    fn();
}
```

如果把这段对 `setTimeout` 的定义放在转化后的代码里面，它的效果将和没有 `setTimeout` 一样，这可以检测究竟是转化的错误，还是由于 `setTimeout` 的引入导致状态的混乱而引起的错误。

### `break` 和 `continue` （究级转化公式）

`break` 和 `continue` 只会出现在 `_body_`中，必须对它们进行处理，因为在很可能导致转化后的语法错误。对前面的公式稍作修改，加入 `breakLoop` 和 `continueLoop` 两个方法：

```js
function loop() {
    if (_cond_) {
        _body_;
        continueLoop();
    } else {
        breakLoop();
    }
}
function continueLoop() {
//  _adj_; // for
    setTimeout(loop, ms);
}
function breakLoop() {
    _postloop_;
}
_preloop_;
//_init_; // for
//_body_; // do...while
loop();
```

再把 `_body_` 改成方法调用：

```js
function loopBody() {
    _body_; // break 改 return false; continue 改 return;
}
```

并在调用 `loopBody` 的时候根据返回值是否 `false` 来决定调用 `breakLoop` 还是 `continueLoop`。于是，就有了以下的公式：

```js
function loop() {
    if (_cond_) {
        if (loopBody() === false) {
            breakLoop();
        } else {
            continueLoop();
        }
    } else {
        breakLoop();
    }
}
function continueLoop() {
//  _adj_; // for
    setTimeout(loop, ms);
}
function breakLoop() {
    _postloop_;
}
function loopBody() {
    _body_; // 在break处return false; continue处return;
}
_preloop_;
//_init_; // for
/*do...while
if (loopBody() === false) {
    breakLoop();
} else {
    continueLoop();
}*/
loop(); // 非 do...while 使用本行代码 do...while 不使用！
```

为了避免 `do_while` 对问题分析的影响，把问题分为两个部分：`for/while` 和 `do_while`。

> NOTE：`break label;` 在 JavaScript 里相当于 goto，实在太复杂了，而且不推荐，所以这里不做讨论。

## 双重 循环

双重循环比单重循环要复杂一些。

一般来说，外层循环的开始将导致内层循环的开始，而内层循环的结束又是外层循环的开始，即外层循环是由内层循环控制的。
双重循环的转化过程中，可以采用由外而内的顺序进行转化（也可以反着来）。

由于双重循环可以是 `for/while/do_while` 的各种组合，比较复杂，所以，就不整理公式了。关键是要知道如何分解循环。
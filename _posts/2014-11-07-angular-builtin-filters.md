---
layout: post
title:  "angular的内建filter"
date:   2014-11-07 22:12:51
categories: angular
---

# 译序

在<http://scotch.io>上看到一篇文章[All About the Built-In AngularJS Filters](http://scotch.io/tutorials/javascript/all-about-the-built-in-angularjs-filters)。虽然用了近一年的angular，可是因为比较仓促，自己写了很多`filter`，即使知道angular可能自带一些`filter`，也没得机会用。所以看到这篇文章，想想自己走过的弯路，觉得有必要翻译一下，毕竟最近参与了团队的翻译小组，拿这篇短文当个练手也不错。

除了尽可能把原作者的意思表达清楚之外，我会在文章中**强势插入**一些自己的想法。

以下是译文。

---

# 介绍

你可能不知道，那就是AngularJS其实内建了蛮多有用的`filter`。但程序员们还是不断地重复着造轮子的事情，实现那些一直都在那里向你招手而你视而不见的功能，
这有可能是因为这些不足以满足特定的需求，但更多的情况是他们不知道它们的存在。

本文中将逐一介绍AngularJS本身提供的`filter`，它们大多在官方文档中有介绍，但可能缺少实际的案例，所以这里将采用大量的代码实例。

那么来吧！

# 使用filter

`filter`，即过滤器，顾名思义，用于**控制输出数据在视图中的显示**，在HTML中你通过`filter`控制绑定的数据的展现方式，有如下的方式：

<p style="background-color: #FCC; color: #C33; padding: 1em; font-size: 0.8em;">
只要你对linux命令有些许了解，肯定会觉得angular的filter语法很熟悉，没错，angular的语法就是采用了管道符“|”。
</p>

## 基本语法

fdsa

```js
function() {
    alert(1);
}
```

fdsa



```
{{ totalCost | currency }}
```


Chaining Filters

Filters can also be chained, by adding the pipe ( | ) character between each filter so if we wanted to apply multiple filters to a single expression it would look something like:

{{ totalCost | currency | filter2 | filter3 }}
Extended Filters

Finally, filters can be extended even further by supporting arguments, for example:

{{ totalCost | currency:"USD$" }}
It is common practice to apply filters directly to the binding expressions in the HTML views, but you can also apply filters in your controllers and directives as well.

Filters Inside of JavaScript Files

The syntax for applying filters in your JavaScript files will look like this:

$filter('number')(15, 5)
This filter is equivalent to {{ 15 | number:5 }} and both will render the number 15 as string to five decimal places (i.e. 15.00000) in your view.

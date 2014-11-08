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

```
{{ totalCost | currency }}
```
## filter链起来

也可以将filter一个接一个用管道符（|）链起来，这样，前一个filter的输出将作为下一个filter的输入:

<p style="background-color: #FCC; color: #C33; padding: 1em; font-size: 0.8em;">
跟linux命令一样，管道符也可以怎么干。
</p>

```
{{ totalCost | currency | filter2 | filter3 }}
```

## 使用参数增强filter

filter的第一个参数是输入数据，后面的参数是可选的参数，可以使用可选参数增强filter的使用场景（如显示货币的filter支持的币种参数，可以让它不局限于只显示美元，我们也不需要为每个不同的币种创建一个个新的filter）：

```
{{ totalCost | currency:"USD$" }}
```

## JavaScript中使用filter

一般来说，filter都是像前面一样在HTML视图里面用于angular表达式，你也可以在JS代码（在你的`controller`、`directive`及service）里面调用`filter`来得到输出结果。语法如下：

```
$filter("number")(15, 5)
```

以上代码的效果等同于`{{ 15 | number:5 }}`，得到的结果是`15.00000`。

<p style="background-color: #FCC; color: #C33; padding: 1em; font-size: 0.8em;">
虽然这样复用了filter的代码，但两对括号连在一起的形式让人多少有点不舒服。所以，如果是你自己写的filter，我建议可以这样，把你filter的主逻辑放到你的一个util service中，filter只是service对view的一个桥接。在JS中调用service，在HTML中调用filter。
</p>

It’s ok if you don’t fully grasp what we’re doing so far, we are just going over the syntax here – next we’ll walk through the built in filters and how they can improve the presentation of your apps.
你如果对这些不了解，不要紧，现在只是在过语法部分；接下去我们来真正过一遍angular自带的这些帮助我们提升应用的展示效果的filter吧。

String Manipulation – Uppercase and Lowercase

AngularJS comes with prebuilt filters for making a string upper or lower-case. The uppercase and lowercase filters will do what their name implies, either convert a string to all uppercase characters, or convert the string to all lowercase characters.

The simplest way to apply this filter to an expression is to add it directly in the view. Please check out the Codepen below to see the Uppercase and Lowercase filters illustrated.


// JavaScript
app.controller('limits', function($scope){
  $scope.copy = "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua."
  $scope.yelling = "LOREM ISPUM DOLOR SIT AMET, CONSECTETUR ADIPISCING ELIT, SED DO EIUSMOD TEMPOR INCIDIUNT UT LABORE ET DOLORE MAGNA ALIQUA."
})

// HTML
Uppercase:
{{ copy | uppercase }}

Lowercase:
{{ yelling | lowercase }}

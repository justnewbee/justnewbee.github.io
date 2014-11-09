---
layout: post
title:  "angular的内建filter"
date:   2014-11-07 22:12:51
categories: angular
---

译序
===

在<http://scotch.io>上看到一篇文章[《All About the Built-In AngularJS Filters》](http://scotch.io/tutorials/javascript/all-about-the-built-in-angularjs-filters)。虽然用了近一年的angular，可是因为比较仓促，自己写了很多`filter`，即使知道angular可能自带一些`filter`，也没得机会用。所以看到这篇文章，想想自己走过的弯路，觉得有必要翻译一下，毕竟最近参与了团队的翻译小组，拿这篇短文当个练手也不错。

除了尽可能把原作者的意思表达清楚之外，我会在文章中**强势插入**一些自己的想法。

**如何使用本文的代码片段**  

作者展示的代码多少有点拖沓，他用[CodePen](http://codepen.io/)，而我更偏爱[JsBin](http://jsbin.com)。因为是内建filter的演示，除了angular之外，你可以不需要任何JS。所以我把他的源代码精简了一下，只留下HTML部分。你需要做的是：

1. 访问[JsBin](http://jsbin.com)，它直接给了你一个空的HTML页面，你可以在它上面直接开始干这干那了；
2. 使用它左上角的“Add library”按钮，加一个angular的CDN进来（如果被墙，日他娘的g@#$%~...可以试试<https://cdnjs.com>上的，如果再不行...本地有的话，改本地测...）；
3. 在`<html>`（或`<body>`）标签上加`ng-app`（或`data-ng-app`，如果你跟我一样有“看到IDE警告就不舒服症”的话）属性
4. 一个angular应用就好了...就这么简单，你把本文中的测试代码拷到`<body>`里面就能看到效果了（当然，要好看点的话，加些`<br />`或放些`<p>`吧）

在看下去之前，我们来从源码里找找看angular内建了哪些`filter`：

```js
register("currency", currencyFilter);
register("date", dateFilter);
register("filter", filterFilter);
register("json", jsonFilter);
register("limitTo", limitToFilter);
register("lowercase", lowercaseFilter);
register("number", numberFilter);
register("orderBy", orderByFilter);
register("uppercase", uppercaseFilter);
```

<p class="warning">
代码的渲染似乎有些问题... 还在调，所以暂时用_{_{_..._}_}_代替angular的表达式语法
</p>

以下是译文。

---

介绍
===

你可能不知道，那就是AngularJS其实内建了蛮多有用的`filter`。但程序员们还是不断地重复着造轮子的事情，实现那些一直都在那里向你招手而你视而不见的功能，
这有可能是因为这些不足以满足特定的需求，但更多的情况是他们不知道它们的存在。

本文中将逐一介绍AngularJS本身提供的`filter`，它们大多在官方文档中有介绍，但可能缺少实际的案例，所以这里将采用大量的代码实例。

那么来吧！

# 使用filter

`filter`，即过滤器，顾名思义，用于**控制输出数据在视图中的显示**，在HTML中你通过`filter`控制绑定的数据的展现方式，有如下的方式：

<p class="note">
只要你对linux命令有些许了解，肯定会觉得angular的filter语法很熟悉，没错，angular的语法就是采用了管道符“|”。
</p>

## 基本语法

```
_{_{_ totalCost | currency _}_}_
```
### filter链起来

也可以将`filter`一个接一个用管道符`|`链起来，这样，前一个`filter`的输出将作为下一个`filter`的输入:

<p class="note">
跟linux命令一样，管道符也可以怎么干。
</p>

```
_{_{_ totalCost | currency | filter2 | filter3 _}_}_
```

### 使用参数增强filter

`filter`的第一个参数是输入数据，后面的参数是可选的参数，可以使用可选参数增强filter的使用场景（如显示货币的filter支持的币种参数，可以让它不局限于只显示美元，我们也不需要为每个不同的币种创建一个个新的filter）：

```
_{_{_ totalCost | currency:"USD$" _}_}_
```

### JavaScript中使用filter

一般来说，filter都是像前面一样在HTML视图里面用于angular表达式，你也可以在JS代码（在你的`controller`、`directive`及service）里面调用`filter`来得到输出结果。语法如下：

```
$filter("number")(15, 5);
```

以上代码的效果等同于`{{ 15 | number:5 }}`，得到的结果是`15.00000`。

<p class="note">
虽然这样复用了filter的代码，但两对括号连在一起的形式让人多少有点不舒服。所以，如果是你自己写的filter，我建议可以这样，把你filter的主逻辑放到你的一个util service中，filter只是service对view的一个桥接。在JS中调用service，在HTML中调用filter。
</p>

# angular的内建filter介绍

你如果对这些有点含糊，没什么头绪，不要紧，这里只是过一下`filter`的语法；接下去我们来真正过一遍angular自带的filter，以及如何使用它们提升应用的展示效果吧。

## 字符串大小写

`uppercase`和`lowercase`做的事情，已经无需再解释了。

```
{{"LOWERCASE" | lowercase}} ➔ lowercase

{{"uppercase" | uppercase}} ➔ UPPERCASE
```

<p class="note">
之所以不把原文那么多内容译出来，是因为我对这两个filter持保留意见：1). 不是所有的语言（比如我们的汉字）都有大小写转换的需求 2). 这既然是显示的事情，
那就让CSS来干吧，`text-transform: uppercase|lowercase`完全可以胜任这两个filter的工作，而且不会被angular重复执行。
</p>

## 数字与货币

angular处理数字和货币的`number`和`currency`这两个filter还是非常有用的。

`number`提供了小数点精确到第几位（四舍五入）的功能，可以处理小数点显示几位的需求。

```
不加任何参数，直接使用：
_{_{_ 50 | number _}_}_ ➔ 50，等同于没用

补齐小数点后4位：
_{_{_ 50 | number:4 _}_}_ ➔ 50.0000

四舍五入到小数点后2位：
_{_{_ 50.458 | number:2 _}_}_ ➔ 50.46
```

`currency`处理货币的显示，多少显示出angular的闪光点，它的作用是在数字前面加上美元符“$”，同时强制精确到小数点后两位，毕竟是钱，要算清楚。当然，你会说那没啥用，我们用RMB，要“￥”。这没问题，那只是angular默认的行为，记得之前我们说的`filter`可以通过参数扩展自己的功能么？`currency`接受货币符号的参数（实际上，那可以是任何内容）。angular 1.3+之后（1.3.0 beta不行），可以接受第二个参数，可以操控小数点。

```
不加任何参数，直接使用：
_{_{_ 50 | currency _}_}_ ➔ $50.00

我要RMB：
_{_{_ 50.99 | currency:"￥" _}_}_ ➔ ￥50.99

“搞怪”货币（不要在项目里这么干啊，不然i18n就不好处理了）：
_{_{_ 50.999 | currency:"你要还我€：" _}_}_ ➔ 你要还我€：51.00

操控小数点（要求angular 1.3+）：
_{_{_ 59.99 | currency:"£":0 _}_}_ ➔ £60
```

## 日期和时间

如果你觉得前面的`filter`都太小儿科了，那么处理日期和时间的`date`就真的很“amazing”了。为什么说它“amazing”呢，首先，它接受的输入类型可以是`Date`对象、
数字以及任何一种符合ISO 8601标准的日期/时间字符串；其次，它可以以各种输出形式满足你的需求。

不管你需要一个完整的格式如“Thursday, October 19, 2014”，还是只要年份、月份、日, 或者它们之间的各种形式的各种组合，什么短名字，长名字，只要年份的末两位等等，angular都能满足你。
总之，`date`能让原本复杂的日期和时间操作变得很轻松，想要更多地了解`date`这个`filter`，去看看[angular有可能被某匪墙了的文档](https://docs.angularjs.org/api/ng/filter/date)。

<p class="note">

</p>

```
输入字符串不符合标准：
_{_{_ "2014-11-8" | date _}_}_ ➔ 2014-11-8

默认显示：
_{_{_ "2014-11-8" | date _}_}_Nov 8, 2014

带个"fullDate"参数：
_{_{_ "2014-11-08" | date:"fullDate" _}_}_ ➔ Saturday, November 8, 2014

（下面用ISO时间作为输入，通过`new Date().toISOString()`获得）

只要年、月、日、时、分、秒：
_{_{_ "2014-11-08T15:27:09.038Z" | date:'yyyy' _}_}_ ➔ 2014
_{_{_ "2014-11-08T15:27:09.038Z" | date:'yy' _}_}_ ➔ 14
_{_{_ "2014-11-08T15:27:09.038Z" | date:'MM' _}_}_ ➔ 11
_{_{_ "2014-11-08T15:27:09.038Z" | date:'M' _}_}_ ➔ 11
_{_{_ "2014-11-08T15:27:09.038Z" | date:'dd' _}_}_ ➔ 08
_{_{_ "2014-11-08T15:27:09.038Z" | date:'d' _}_}_ ➔ 8
_{_{_ "2014-11-08T15:27:09.038Z" | date:'HH' _}_}_ ➔ 23（不要奇怪，我们是+8区，15+8=23）
_{_{_ "2014-11-08T15:27:09.038Z" | date:'H' _}_}_ ➔ 23
_{_{_ "2014-11-08T15:27:09.038Z" | date:'hh' _}_}_ ➔ 11
_{_{_ "2014-11-08T15:27:09.038Z" | date:'h' _}_}_ ➔ 11
_{_{_ "2014-11-08T15:27:09.038Z" | date:'mm' _}_}_ ➔ 27
_{_{_ "2014-11-08T15:27:09.038Z" | date:'m' _}_}_ ➔ 27
_{_{_ "2014-11-08T15:27:09.038Z" | date:'ss' _}_}_ ➔ 09
_{_{_ "2014-11-08T15:27:09.038Z" | date:'s' _}_}_ ➔ 9

Custom Date Filter:
_{_{_ dateUTC | date:"'Year:' yyyy, 'Month:' MMM, 'Day:' EEEE" _}_}_
```

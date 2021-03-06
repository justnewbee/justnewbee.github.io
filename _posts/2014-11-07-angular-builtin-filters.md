---
layout: post
title: angular 的内建 filter 「译」
date: 2014-11-07 22:12:51
categories: translation
tags: angular
---

译序
===

在 <http://scotch.io> 上看到一篇文章 [《All About the Built-In AngularJS Filters》](http://scotch.io/tutorials/javascript/all-about-the-built-in-angularjs-filters)。虽然用了近一年的 angular，可是因为比较仓促，自己写了很多 filter，即使知道 angular 可能自带一些 filter，也没得机会用。所以看到这篇文章，想想自己走过的弯路，觉得有必要翻译一下，毕竟最近参与了团队的翻译小组，拿这篇短文当个练手也不错。

除了尽可能把原作者的意思表达清楚之外，我会在文章中**强势插入**一些自己的想法。

**如何使用本文的代码片段**  

作者展示的代码多少有点拖沓，他用 [CodePen](http://codepen.io/)，而我更偏爱 [JsBin](http://jsbin.com)。因为是内建 filter 的演示，除了 angular 之外，你可以不需要任何 JS。所以我把他的源代码精简了一下，只留下 HTML 部分。你需要做的是：

1. 访问 [JsBin](http://jsbin.com)，它直接给了你一个空的 HTML 页面，你可以在它上面直接开始干这干那了；
2. 使用它左上角的「Add library」按钮，加一个 angular 的 CDN 进来（如果被墙，日他娘的 g@#$%~... 可以试试 <https://cdnjs.com> 上的，如果再不行...本地有的话，改本地测...）；
3. 在 `<html>`（或 `<body>`）标签上加 `ng-app`（或 `data-ng-app`，如果你跟我一样有「看到IDE警告就不舒服症」的话）属性
4. 一个 angular 应用就好了... 就这么简单，你把本文中的测试代码拷到 `<body>` 里面就能看到效果了（当然，要好看点的话，加些 `` 或放些 `<p>` 吧）

<p class="warning">
代码的渲染似乎有些问题... 还在调，所以暂时用_{_{_..._}_}_代替angular的表达式语法
</p>

以下是译文。

---

介绍
===

你可能不知道，那就是 AngularJS 其实内建了蛮多有用的 filter。但程序员们还是不断地重复着造轮子的事情，实现那些一直都在那里向你招手而你视而不见的功能，
这有可能是因为这些不足以满足特定的需求，但更多的情况是他们不知道它们的存在。

本文中将逐一介绍 AngularJS 本身提供的 filter，它们大多在官方文档中有介绍，但可能缺少实际的案例，所以这里将采用大量的代码实例。

那么来吧！

# 使用 filter

filter，即过滤器，顾名思义，用于**控制输出数据在视图中的显示**，在 HTML 中你通过 filter 控制绑定的数据的展现方式，有如下的方式：

<p class="note">
只要你对 linux 命令有些许了解，肯定会觉得 angular 的 filter 语法很熟悉，没错，angular 的语法就是采用了管道符 `|`。
</p>

## 基本语法

```html
_{_{_ totalCost | currency _}_}_
```

### filter 链起来

也可以将 filter 一个接一个用管道符 `|` 链起来，这样，前一个 filter 的输出将作为下一个 filter 的输入:

<p class="note">
跟 linux 命令一样，管道符也可以怎么干。
</p>

```html
_{_{_ totalCost | currency | filter2 | filter3 _}_}_
```

### 使用参数增强 filter

filter 的第一个参数是输入数据，后面的参数是可选的参数，可以使用可选参数增强 filter 的使用场景（如显示货币的 filter 支持的币种参数，可以让它不局限于只显示美元，我们也不需要为每个不同的币种创建一个个新的 filter）：

```
_{_{_ totalCost | currency:"USD$" _}_}_
```

### JavaScript 中使用 filter

一般来说，filter 都是像前面一样在 HTML 视图里面用于 angular 表达式，你也可以在 JS 代码（在你的 controller、directive 及 service）里面调用 filter 来得到输出结果。语法如下：

```js
$filter("number")(15, 5);
```

以上代码的效果等同于 `{{ 15 | number:5 }}`，得到的结果是 `15.00000`。

<p class="note">
虽然这样复用了 filter 的代码，但两对括号连在一起的形式让人多少有点不舒服。所以，如果是你自己写的 filter，我建议可以这样，把你 filter 的主逻辑放到你的一个 service 中，filter 只是 service 的一个桥接。在 JS 中调用 service，在 HTML 中调用 filter。
</p>

# angular 的内建 filter 介绍

你如果对这些有点迷糊，没什么头绪，不要紧，这里只是过一下 filter 的语法；接下去我们来真正过一遍 angular 自带的 filter，以及如何使用它们提升应用的展示效果吧。

## 字符串大小写

filter `uppercase` 和 `lowercase` 做的事情，已经无需再解释了。

```
{{"LOWERCASE" | lowercase}} ➔ lowercase

{{"uppercase" | uppercase}} ➔ UPPERCASE
```

<p class="note">
对这两个 filter，我不是很赞同：1). 不是所有的语言（比如我们的汉字）都有大小写转换的需求 2). 这既然是显示的事情，那就让 CSS 来干吧，`text-transform: uppercase|lowercase` 完全可以胜任这两个 filter 的工作，而且不会被 angular 重复执行。
</p>

## 数字与货币

filter `number` 和 `currency` 分别处理数字和货币事宜，还是非常有用的。

`number` 提供了小数点精确到第几位（四舍五入）的功能，可以处理小数点显示几位的需求。

```
不加任何参数，直接使用：
_{_{_ 50 | number _}_}_ ➔ 50，等同于没用

补齐小数点后4位：
_{_{_ 50 | number:4 _}_}_ ➔ 50.0000

四舍五入到小数点后2位：
_{_{_ 50.458 | number:2 _}_}_ ➔ 50.46
```

`currency` 处理货币的显示，多少显示出 angular 的闪光点，它的作用是在数字前面加上美元符「$」，同时**强制精确**到小数点后两位，毕竟是钱，要算清楚。当然，你会说那没啥用，我们用 RMB，要「￥」。这没问题，那只是 angular 默认的行为，记得之前我们说的 filter 可以通过参数扩展自己的功能么？`currency` 接受货币符号的参数（实际上，那可以是任何内容）。angular 1.3+ 之后（1.3.0 beta 不行），可以接受第二个参数，可以操控小数点。

```
不加任何参数，直接使用：
_{_{_ 50 | currency _}_}_ ➔ $50.00

我要RMB：
_{_{_ 50.99 | currency:"￥" _}_}_ ➔ ￥50.99

无节操货币（不要在项目里这么干啊，不然i18n就不好处理了）：
_{_{_ 50.999 | currency:"你要还我€：" _}_}_ ➔ 你要还我€：51.00

操控小数点（要求angular 1.3+）：
_{_{_ 59.99 | currency:"£":0 _}_}_ ➔ £60
```

## 日期和时间

如果你觉得前面的 filter 都太小儿科了，那么处理日期和时间的 `date` 就真的很「amazing」了。为什么说它「amazing」呢，首先，它接受的输入类型可以是 `Date` 对象、
数字以及任何一种符合ISO 8601标准的日期/时间字符串；其次，它可以以各种输出形式满足你的需求。

不管你需要一个完整的格式如「Thursday, October 19, 2014」，还是只要年份、月份、日, 或者它们之间的各种形式的各种组合，什么短名字，长名字，只要年份的末两位等等，angular 都能满足你。
总之，`date` 能让原本复杂的日期和时间操作变得很轻松，想要更多地了解 `date` 这个 filter 及其支持的各种格式，去看看 [angular有可能被某匪墙了的文档](https://docs.angularjs.org/api/ng/filter/date)。

```
输入字符串不符合标准：
_{_{_ "2014-11-8" | date _}_}_ ➔ 2014-11-8

默认显示：
_{_{_ "2014-11-8" | date _}_}_ ➔ Nov 8, 2014

带个"fullDate"参数：
_{_{_ "2014-11-08" | date:"fullDate" _}_}_ ➔ Saturday, November 8, 2014

（下面用ISO时间作为输入，通过 `new Date().toISOString()` 获得）

只要年`y`、月`M`、日`d`、周几`E`、时`hH`（对输入15，输出23，不要奇怪，我们是+8区，15+8=23）、上下午`a`、分`m`、秒`s`：
_{_{_ "2014-11-08T15:27:09.038Z" | date:"yyyy" _}_}_ ➔ 2014
_{_{_ "2014-11-08T15:27:09.038Z" | date:"yy" _}_}_ ➔ 14
_{_{_ "2014-11-08T15:27:09.038Z" | date:"y" _}_}_ ➔ 2014
_{_{_ "2014-11-08T15:27:09.038Z" | date:"MMMM" _}_}_ ➔ November
_{_{_ "2014-11-08T15:27:09.038Z" | date:"MMM" _}_}_ ➔ Nov
_{_{_ "2014-11-08T15:27:09.038Z" | date:"MM" _}_}_ ➔ 11
_{_{_ "2014-11-08T15:27:09.038Z" | date:"M" _}_}_ ➔ 11
_{_{_ "2014-11-08T15:27:09.038Z" | date:"dd" _}_}_ ➔ 08
_{_{_ "2014-11-08T15:27:09.038Z" | date:"d" _}_}_ ➔ 8
_{_{_ "2014-11-08T15:27:09.038Z" | date:"EEEE" _}_}_ ➔ Saturday
_{_{_ "2014-11-08T15:27:09.038Z" | date:"EEE" _}_}_ ➔ Sat
_{_{_ "2014-11-08T15:27:09.038Z" | date:"HH" _}_}_ ➔ 23
_{_{_ "2014-11-08T15:27:09.038Z" | date:"H" _}_}_ ➔ 23
_{_{_ "2014-11-08T15:27:09.038Z" | date:"a" _}_}_ ➔ PM
_{_{_ "2014-11-08T15:27:09.038Z" | date:"hh" _}_}_ ➔ 11
_{_{_ "2014-11-08T15:27:09.038Z" | date:"h" _}_}_ ➔ 11
_{_{_ "2014-11-08T15:27:09.038Z" | date:"mm" _}_}_ ➔ 27
_{_{_ "2014-11-08T15:27:09.038Z" | date:"m" _}_}_ ➔ 27
_{_{_ "2014-11-08T15:27:09.038Z" | date:"ss" _}_}_ ➔ 09
_{_{_ "2014-11-08T15:27:09.038Z" | date:"s" _}_}_ ➔ 9

利用前面的各种格式自定义格式（这里的输入是前面的毫秒数，里面的英文字串不是格式，但含有格式的字符如 "a"，加上引号可以原文输出）：
_{_{_ 1415460429038 | date:"yyyy-MMM-dd EEEE" _}_}_ ➔ 2014-Nov-08 Saturday
_{_{_ 1415460429038 | date:"'Year:' yyyy, 'Month:' MMM, 'Day:' EEEE" _}_}_ ➔ Year: 2014, Month: Nov, Day: Saturday
_{_{_ 1415460429038 | date:"yy年M月d日" _}_}_ ➔ 14年11月8日
```

## JSON

在 angular 里，如果表达式里面是个对象，它会以 JSON 的方式输出，这一般用于 debug。但如果是个比较复杂的对象，输出来的对像基本上是无法看清的，所以就有了 `json` 这个 filter。
它做的事情很简单，只是把对象以好看的形式输出来，也不接受参数。唯一要做的是，你需要输出到 `<pre>` 里面，不然 HTML 忽略换行与空格，还是一样看不清。可以说，没有 `<pre>`，`json` 的作用无法体现。

```html
<pre data-ng-init="o = {name: 'Bob', email: 'bob@inbox', password: 'youshallnotpass',
        activities: ['jogging', 'swimming', 'boxing'],
        eligibility: true, hasActiveDevice: false}">
_{_{_ o | json _}_}_
</pre>

得到的输出如下：
{
  "name": "Bob",
  "email": "bob@inbox",
  "password": "youshallnotpass",
  "activities": [
    "jogging",
    "swimming",
    "boxing"
  ],
  "eligibility": true,
  "hasActiveDevice": false
}
```

## 长度限制

`limitTo`，又是一个顾名思义的 filter，帮你把字符串或者数组的长度限定在一个范围之内。例如，对一个长度为 15 的字符串使用 `limitTo:10` 将只显示该字符串的前面 10 个字符。

`limitTo` 用于数组的时候，跟 `ng-repeat` 一起搭配会很强大，你可以很轻松地利用它们在你的应用上搭建一个分页系统。

`limitTo` 的一个常用场景是文本预览，比方说在 blog 的首页上，每篇 blog 你想只显示 250 个字作为预览，你只需要简单地 `_{_{_ previewCopy | limitTo: 250 _}_}_` 就可以了。

<p class="note">
下面的例子中用了大量的「lorem...」，这个是国外程序员很喜欢用的范例文本，很多编辑器都有 snippet，起个头「lorem」，然后按一下 `tab`，整段文本就出来了。如果编辑器本身没有这个，装个 [emmet 插件](http://emmet.io/)。
</p>

```
_{_{_ "Lorem ipsum dolor sit amet, consectetur adipisicing elit. Illo consequatur quas perferendis consequuntur dolor? Molestias molestiae dolor illum nobis sit quis vitae beatae excepturi quisquam ullam sint, enim, laboriosam voluptatibus." | limitTo:12 _}_}_ ➔ Lorem ipsum dol
```

以上代码等价于 `_{_{_ str.substring(0, 15) _}_}_`，体现不出 `limitTo` 的强大之处，最多的时候还是用在数组上：

```html
<ul data-ng-init="arr = [1,2,3,4,5,6,7,8,9,10]">
    <li data-ng-repeat="_v in arr | limitTo:3">_{_{_ _v _}_}</li>
</ul>
```

<p class="note">
    <code>limitTo</code> 还可以接负数，表示对字符串和数组倒着截取。但 1.0.x 的对字符串会忽略负数，不做截取。
</p>

# 总结

到这里，我们已经看过了 angular 所有内建的 filter，它们当中有简单到可以不存在的大小写转换，也有复杂而功能强大的日期操作。同时我们也了解了使用 filter 的几种不同的方式，最常用的是应用于 HTML 的表达式中，还有如何在 JS 中中间接使用 filter。

希望本文可以帮你了解 filter 的工作机制，因为接下来，我们将继续 angular 的 filter 之旅，自己写 filter。Stay tuned 吧！

<hr />

以上是译文。

译后
===

作者写的不错，不过也有疏漏。在原文的评论中也有些不错的意见，所以，接下去我们再做些简单地研究。

在看下去之前，我们来从源码里找找看 angular 内建了哪些 filter：

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

所以，还有 `filter` 和 `orderBy` 这两个 filter 没给介绍到，所以这里补个漏。不过在继续之前，我们再统一回顾一下语法：

```
_{_{_ currency_expression | currency : symbol : fractionSize _}_}_
_{_{_ date_expression | date : format : timezone _}_}_
_{_{_ filter_expression | filter : expression : comparator _}_}_
_{_{_ json_expression | json _}_}_
_{_{_ limitTo_expression | limitTo : limit _}_}_
_{_{_ lowercase_expression | lowercase _}_}_
_{_{_ uppercase_expression | uppercase _}_}_
_{_{_ number_expression | number : fractionSize _}_}_
_{_{_ orderBy_expression | orderBy : expression : reverse} _}_}_
```

# filter filter

`filter` filter 应用于数组，如果你知道数组 `Array` 本身有个 `filter` 方法，你就知道它的工作是什么。`filter` 功能强大，参数多变，所以需要多花写篇幅好好捋一捋。这是 `filter` 的用法：

```
_{_{_ array | filter : expression : comparator _}_}_
```

## expression

与数组的 `filter` 不同，`filter` filter 可以接受两个参数：`expression` 和 `comparator`，后者是可选的，我们先看前者，它有三种选择：

* 字符串 `"value"`
* 对象 `{key: "value"}`
* 函数 `function(v, k) { ... }`

**expression为字符串**  
对数组里的每个元素的值（本身的值或内部元素的值）`toString` 后进行模糊匹配：

<p class="note">
对象/数组，这种匹配是递归的，所以不论匹配隐藏地多深，都能够被挖掘出来，你也可以认为，此操作会把对象扁平化进行匹配，即 `{a: {b: {c: { d: { e: 'deep 1ns1de'}}}}}` 扁平成 `{'a.b.c.d.e': 'deep 1ns1de'}`，用过 mongo 的人对此应该很熟悉。
</p>

```html
<ul data-ng-init="arrHybrid = [71, 'very2', 'he110', {name: 'not_1'}, {a: {b: {c: { d: { e: 'deep 1ns1de'}}}}}, {foo1: 'bar'}, [1, 12, 123, 1234]]">
  <li data-ng-repeat="_v in arrHybrid | filter:'1'">_{_{_ _v _}_}</li>
</ul>
```

输出如下：

```
71
he110
{"name":"not_1"}
{"a":{"b":{"c":{"d":{"e":"deep 1ns1de"}}}}}
[1,12,123,1234]
```

**expression为对象**  
匹配所有的 expression 中的所有 key-value 对，每个匹配的方式跟之前的一样。还是上面的例子，改为 `filter:{a:'1'}` 或 `filter:{'a.b':'1'}`，则输出结果：

```
{"name":"not_1"}
```

但如果是 `{'a.c': 1}`（找不到 key）或 `{a: '1', name: '1'}`（没有两个都符合的），将没有任何输出。

还可以用特殊的 `$` 作为 key，`filter:{$: 1}` 等效于 `filter:1`。

需要注意的是，字符串和数组也可以看成是以数字为key的对象，`filter:{3: 1}` 将得到如下输出：

```
he110
[1,12,123,1234]
```

<p class="warning">
`expression`中的匹配字串前面加 `!` 可以对结果取反，可以如果就是要匹配「!」呢？我没看到...
</p>

**expression为方法**  
跟 `Array` 的方式一样，它可以让我们做更多地事情，更精确的匹配，你可以想象成上面的两种方式是这种的特例，angular 在内部为你实现了 `filter` 的方法。

假设我们需要把所有类型为 `String` 的元素 filter 出来，需要在 JsBin 里加一点 JS（因为用了 JS，所以把原本 `ng-init` 对 `arrHybrid` 的初始化也拿过来了）：

```js
function FilterTestCtrl($scope) {
    angular.extend($scope, {
        arrHybrid: [71, "very2", "he110", {
            name: "not_1"
        }, {
            a: {
                b: {
                    c: {
                        d: {
                            e: "deep 1ns1de"
                        }
                    }
                }
            }
        }, {
            foo1: "bar"
        }, [1, 12, 123, 1234]],
        
        filterFn: function(v, k) {
            return angular.isString(v);
        }
    });
}
```

修改 HTML，加上 `ng-controller`：

```html
<ul ng-controller="FilterTestCtrl">
    <li data-ng-repeat="_v in arrHybrid | filter:filterFn">{{ _v }}</li>
</ul>
```

输出如下：

```
very2
he110
```

## comparator

`comparator` 可以对每个匹配的
Comparator which is used in determining if the expected value (from the filter expression) and actual value (from the object in the array) should be considered a match.

Can be one of:

function(actual, expected): The function will be given the object value and the predicate value to compare and should return true if the item should be included in filtered result.

true: A shorthand for function(actual, expected) { return angular.equals(expected, actual)}. this is essentially strict comparison of expected and actual.

false|undefined: A short hand for a function which will look for a substring match in case insensitive way.


STILL WORKING...

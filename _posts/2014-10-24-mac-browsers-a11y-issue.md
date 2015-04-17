Mac上浏览器a11y陷阱
===

# 1 序

## 1.1 a11y何物

**_a11y_**，即accessibility，专门指代计算机软件领域的用户可访问性。计算机软件需要在设计和实现上让残障人士，尤其是视力有问题的人，也能够明确地知道自己目前所处的位置，以及可以进行什么样的操作。

其实，对于正常人而言，a11y也很重要，比如有些人喜欢使用键盘做一些原本需要用鼠标的操作，因为那样快，而且看起来会很牛逼的样子。

a11y是个很大很复杂的话题，这里只讨论网页中最最基础的a11y需求——使用键盘进行导航和操作。

## 1.1 tabIndex

当你在网页中按TAB键，会发现网页上某个元素获得了焦点；这时如果你按住TAB不放，这个焦点会依次往后传递给下一个能接受焦点的元素，那么这些依次经过的元素就组成一个**_TAB流（TAB flow）_**。

HTML提供了一个叫作`tabIndex`的属性告诉浏览器，这个元素是否可以获得焦点，在获得焦点的时候又是如何一个顺序。一般来说，所有的表单元素、按钮和链接（带`href`的`a`元素）都_应该_可以获得焦点的；而像`<form>`、`<div>`、`<span>`、`<p>`、`<ul>`、`<li>`则无法获得焦点。

[W3C规定了哪些元素能被获得焦点](http://www.w3.org/TR/html401/interact/forms.html#adef-tabindex)：

> The following elements support the [`tabindex`](http://www.w3.org/TR/html401/interact/forms.html#adef-tabindex) attribute: [A](http://www.w3.org/TR/html401/struct/links.html#edef-A), [AREA](http://www.w3.org/TR/html401/struct/objects.html#edef-AREA), [BUTTON](http://www.w3.org/TR/html401/interact/forms.html#edef-BUTTON), [INPUT](http://www.w3.org/TR/html401/interact/forms.html#edef-INPUT), [OBJECT](http://www.w3.org/TR/html401/struct/objects.html#edef-OBJECT), [SELECT](http://www.w3.org/TR/html401/interact/forms.html#edef-SELECT), and [TEXTAREA](http://www.w3.org/TR/html401/interact/forms.html#edef-TEXTAREA).

`tabIndex`的取值会产生如下效果：

1. 负数（一般用`-1`）：表示元素不会出现在TAB流，但点击后可获得焦点；
2. `0`：元素跟原生的表单元素一样，既能获得焦点，又会出现在TAB流中；
3. 正数：用于调整tab的优先级，越小的优先级越高。

也就是说，在TAB流中的元素一定是可以获得焦点的，而能获得焦点的元素则不一定出现在TAB流中。

	注意：在HTML（XHTML除外）中的`tabIndex`属性，大小写是无关的，可以写成任意鬼样子，如"tAbInDEX"；但在JavaScript中，则必须使用`tabIndex`。浏览器为每个元素设了默认的`tabIndex`，可想而知，表单元素、按钮及连接的`tabIndex`值默认为`0`；有趣的是，当你去看那些不能获取焦点的元素的时候，会发现`tabIndex`值居然是`-1`！莫慌，用`hasOwnProperty`检查一下就会发现这个`-1`不是它本身所有的，所以要真正让`tabIndex`生效，必须用HTML或者JS的方式，让它真正获得自己own的`tabIndex`属性。

参考：[Making Elements Focusable With TabIndex](http://snook.ca/archives/accessibility_and_usability/elements_focusable_with_tabindex)

# 2. 网页的键盘操作

一般用户在访问网页的时候会同时使用键盘和鼠标，而盲人用户一般就只能使用键盘的某些键（TAB，SPACE，ENTER以及方向键）来进行导航。

TAB的作用是让某个元素获得焦点；SPACE和ENTER在有焦点元素的前提下，可以代替鼠标的点击；方向键的作用是在一个局限的范围内进行导航，比如在`select`内部选择`option`。

如果要让用户只用键盘，是不是仅仅使用浏览器自带的一些a11y的解决方案就行了呢？

假设用户需要访问一个表单（`<form>`），表单（原生的）元素包括有：

* `input`（`:text`、`:hidden`、`:radio`、`:checkbox`、`:submit`、`:reset`、`:button`、`:number`、`:date`、`:url`、`:range`）
* textarea
* select
* button

当然，表单里面可能有一些链接（`<a>`元素）；也有一些自定义的JS控件，可能需要用户点击操作。

接下来我们通过几个案例来分析（若不做特别说明，浏览器所处的平台为_Mac OSX 10.10_）。

## 2.1 TAB导航

键盘访问页面元素是一切的开端，我们使用TAB选择下一个或上一个元素。

### 2.1.1 案例

测试bin：<http://jsbin.com/vatuv>。

这里包含了典型的表单元素，以及一个`tabIndex`设置为`0`的`<p>`（权当它是一个JS控件）。由于有些浏览器对某些元素取消了获焦时的`outline`，例子中默认加了`outline`，可以用最上面的复选框来取消。

### 2.1.2 发现问题

* **Safari 7** 跳过的元素最多，`input:range`、`input:radio`、`input:checkbox`、`a`、`button`，但却可以聚焦到`tabIndex`为`0`的`<p>`。
* **Firefox 33** 跳过`a`（Window的Firefox不会）
* **Chrome 39** OK
* **Opera 25** 跳过`a`
* **IE 10（Win8）** OK

我原以为这是Safari的一个很严重的bug，Google之后发现，这其实是它的一个feature：Mac Safari默认TAB到“重要的”表单元素，如果想TAB到其它元素，需要按`Option` + `TAB`。Window平台没有`Option`，所以Windows上的Safari表现完全不一样，其结果跟Mac上的Firefox一样。但对于HTML标准来说，这的确是个bug。

本节参考：

* [Enabling keyboard navigation in Mac OS X Web browsers](http://www.456bereastreet.com/archive/200906/enabling_keyboard_navigation_in_mac_os_x_web_browsers/)
* [Results of browser testing for when elements take the focus](http://jspro.brothercake.com/focus/results.html)（但似乎跟我在Mac上的测试结果有些出入）

### 2.1.3 解决问题

Safari通过组合键`Option` + `TAB`可以TAB到所有的元素，但`a`元素在Firefox和Opera下却还是不行。不能TAB到，即不能获得焦点，意味着不能被用户感知，更不用说跟它进行交互，如何解决呢？

Opera可以通过为`a`手动添加`tabIndex`来解决；但Firefox不行...继续Google之后，发现了代码做不到的两个解决方案。

方法一：调教浏览器  
浏览器地址栏输入`about:config`，添加或修改`accessibility.tabfocus`项，整型（字符串类型无效），值为`7`。Windows的Firefox中默认值为`7`，如果改成`3`就跟Mac下一个德行了；但Mac下却根本没这个项，这大概就是这个问题的根本原因所在。

方法二：调教系统  
如果不改Firefox的`about:config`，还有一个办法就是System Preferences > Keyboard > Shortcuts，最下面选择“All controls”：

![Mac系统设定](/images/mac_setting_keyboard.png)

系统的，即是大家的，那么它对Safari和Opera是否同样有效呢？答案是：有点效果，只是有点...Safari上原本无法TAB到的`button`、`radio`和`checkbox`等都可以了，但`a`还是不行，即使加`tabIndex`也不行，还是必须`Option` + `TAB`；Opera仍旧只能靠`tabIndex`。看来，这个系统设定是为Firefox量身定制的，真是“百撕不得骑姐”。

本节参考：

* [Tabbing problems in Firefox in Mac OS X](http://robertnyman.com/2007/01/18/551/)

### 2.1.4 总结陈词

让`a`获得焦点的只能靠组合大招：Safari让用户按`Option` + `TAB`；Firefox让用户改config；Opera加`tabIndex`。

我们能做的也只有加`tabIndex`了，所以尽量不要用`a`做JS控件的主节点了，用`div`或`span`加`tabIndex="0"`比较靠谱。

## 2.2 ENTER/SPACE点击

元素获得焦点之后，如果是可以点击的，则希望用`ENTER`和`SPACE`来操作。

### 2.2.1 案例

测试bin：<http://jsbin.com/zipiqi>。

在之前的案例上做了一点点改动，为最后面的`a`、`button`和`p`加了`onclick`。

### 2.2.2 发现问题

* **Safari 7** （`Option` + `TAB`）`a`只对`ENTER`有反应；`button`接受`ENTER`和`SPACE`；而`p`，看下去就知道哪哪都不行；
* **Firefox 33** `a`接受`ENTER`（若不解决`TAB`跳过问题，必须通过别的方式使其获焦，如使用Firefox的搜索框选中它后按`ESC`）；`button`接受`ENTER`和`SPACE`；`p`不行；
* **Chrome 39** `a`接受`ENTER`；`button`接受`ENTER`和`SPACE`；`p`不行；
* **Opera 25** `a`接受`ENTER`（若不加`tabIndex`，似乎很难获得焦点，从而失败）；`button`接受`ENTER`和`SPACE`；`p`不行；
* **IE 10（Win8）** `a`接受`ENTER`；`button`接受`ENTER`和`SPACE`；`p`不行；

### 2.2.3 解决问题

不是所有获得焦点的元素都会在`ENTER`**以及**`SPACE`的时候执行click事件。

悲催的是，HTML并没有提供任何属性告诉浏览器，“把我当button一样蹂躏吧”，所以即使对某个元素设置了`role="button"`，也无法让它自己接受`ENTER`和`SPACE`。幸运的是，这是代码可以解决的。既然，一个元素获得了焦点，那它就能绑定键盘事件（`keyup`、`keydown`、`keypress`），需要注意的是有些按键会触发浏览器的默认事件，比如`SPACE`导致浏览器滚动，需要`preventDefault`一下。

### 2.2.4 总结陈词

这里只需要一点点JS绑定事件的技巧即可。

参考：

* [Making elements keyboard focusable and clickable](http://www.456bereastreet.com/archive/201302/making_elements_keyboard_focusable_and_clickable/)
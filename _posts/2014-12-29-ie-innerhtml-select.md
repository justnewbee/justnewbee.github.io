---
layout: post
title: IE 下对 select 元素执行 innerHTML 操作的问题
date: 2014-12-29 20:19:41
categories: frontend
tags: ie, html, js
---

今天被报一个 IE6-8 上的 bug，代码不是我写的，但我是维护者。

# 问题描述和分析

bug 说的是 IE6-8 下，原本应该有选项的几个 `<select>` 元素，点击后拉下来的框里空空如也。这些 `<select>` 应该至少有个「请选择」之类的文本，但什么都没有，导致用户行为无法继续。

页面发生问题处，有四个 `<select>` 元素，前面三个出现了问题，最后一个却是好的；刷新发现有一秒不到的时间，四个都是好的，所以基本上可以确定是 JS 造成的。查了一下源代码，确实对这几个 `<select>` 有JS操作，KISSY 的 `append` 操作。

因为是 IE6，所以先用 [DebugBar](http://www.debugbar.com/) 看了一下出问题的 `<select>` 的结构，发现里面是有内容的，但不是正常的 `option` 列表，在 DOM 上也多出几个单独的 `</option>` 元素（可以看后面的截图）。

# 问题的隔离和解决

对 KISSY 并不是特别了解，先不研究它的具体实现，猜测最终用的是 `innerHTML`。要验证一个猜想是否成立，最重要的是排除一切其他干扰，所以使用如下最干净的代码进行测试：

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8" />
<script>
function testInnerHTML() {
    var sel = document.getElementById("test");
    sel.innerHTML = sel.innerHTML;
}
</script>
</head>
<body>
<select id="test">
    <option value="">-- please select --</option>
    <option value="1">option 1</option>
    <option value="2">option 2</option>
</select>
<button onclick="testInnerHTML()">innerHTML</button>
</body>
</html>
```

测试后果不出所料，对 `<select>` 直接进行 `innerHTML` 操作果然有问题，可以看一下正常与非正常的区别：

![正常的select与出问题的select](/images/posts/html_select_after_innerhtml.jpg "正常的select与出问题的select")

而且，整个 `<select>` 元素的内容也变得很奇怪了（第一个起始 `<option>` 不知跑哪去了）：

```
<SELECT id=test>
    -- please select --</OPTION>
    <OPTION value="option1">
    option 1
    </OPTION>
    <OPTION value="option2">
    option 2
    </OPTION>
</SELECT>
```

既然抓住了症结，修它自然不费什么事，果断用 `new Option(text, value)`，不过这之前还需要小小验证一把：

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8" />
<script>
function testInnerHTML() {
    var sel = document.getElementById("test");
    sel.innerHTML = sel.innerHTML;
}
function testNewOption() {
    var sel = document.getElementById("test"),
        arr = [{
            v: "1",
            t: "option 1"
        }, {
            v: "2",
            t: "option 3"
        }];
    
    sel.options.length = 0; // clear
    sel.options[0] = new Option("-- please select --", "");
    for (var i = 0, l = arr.length; i < l; i++) {
        // cannot use push cause options is not array
        sel.options[sel.options.length] = new Option(arr[i].t, arr[i].v);
    }
}
</script>
</head>
<body>
<select id="test">
    <option value="">-- please select --</option>
    <option value="option1">option 1</option>
    <option value="option2">option 2</option>
</select>
<button onclick="testInnerHTML()">innerHTML</button>
<button onclick="testNewOption()">new Option(text, value)</button>
</body>
</html>
```

结果自然是绝对 OK 的。

# jQuery

我自然而然地想到了 jQuery，它的几个接口是否也有问题呢，试了一下 `html`、`append`，答案是 OK。所以，KISSY 还是有很多问题要修复的。
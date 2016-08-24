---
layout: post
title: IE6-9 下 required 属性兼容性问题
date: 2015-04-15 10:49:51
categories: frontend
tags: html, js, browser
---

在使用 HTML5 的 `required` 属性时，碰到了 IE6-9 的兼容性问题，这里记录一下。

# 问题

具体的细节这里就不说了，最终的问题是使用 KISSY 的 `prop("required")` 无法得到想要的布尔值。

# 分析

获取 `required` 值有以下几种方式：

1. KISSY/jQuery `$node.prop("required")` 「最佳」实践  _期望_得到 `true/false`
2. KISSY/jQuery `$node.attr("required")` 但只能得到 `String`
3. 直接 `node.required` 逼不得已的做法
4. 正则匹配 `/\brequired\b/i.test(node.outerHTML)` 最最逼不得已的做法

在 JsBin 上写了一个 bin 用来测试：<http://jsbin.com/rayuqe>。

以下是 bin 里用到的单选按钮，测试脚本会在每个按钮后面输出它的 `outerHTML`，`prop("required")`，`attr("required")` 和 `.required`：

```html
<!-- 这里前三种 required 是正确的写法
后面三个 true/fals/fuck 都是不正确的 但浏览器照样认为那是 required -->
<input type="radio" />
<input type="radio" required />
<input type="radio" required="" />
<input type="radio" required="required" />
<input type="radio" required="true" />
<input type="radio" required="false" />
<input type="radio" required="fuck" />
```

# 现象

以下是 IE 不同版本下测试结果（该结论在 Win10 虚拟机的 IE11 中得到）：

## IE11/10

这两个版本的标准化已经相当不错了，可以愉快地一起玩耍了，使用 `prop("required")` 可以获得正确的布尔值。

![IE 11](/images/posts/required_in_ie11.jpg)
![IE 10](/images/posts/required_in_ie10.jpg)

## IE9

不是我什么都不想要，而是她什么都不想给。这里有一点可以看到，jQuery 的 `attr()` 方法强过 KISSY。但如果用的是 KISSY，只能降级到最最不想用的正则匹配了。

![IE 9](/images/posts/required_in_ie9.jpg)

## IE 8/7

`prop("require")` 返回的是字符串类型，KISSY 的 `attr("require")` 可能返回 `undefined`，所以如果要正确地获得 `required` 为布尔值的话，可以采用 `!!propRequired || propRequired === ""`。

![IE 8](/images/posts/required_in_ie8.jpg)
![IE 7](/images/posts/required_in_ie7.jpg)

# 解决问题

总结起来，要获取某 `input` 的 `required` 值，需要被逼无奈地使用 UA 检测，以下是使用 KISSY 的代码片段：

```
...
if (UA.ie && UA.ie === 9) {
	return /\brequired\b/i.test(input.outerHTML);
}

var required = $input.prop("required");
return !!required || required === "";// required === "" for IE6-8
```

而使用 jQuery 的话，可以这样：

```
...
return !!$input.attr("required");
```

# 展望

当然这都不是我们想要的，我们需要的是 KISSY 和 jQuery 可以帮我们 cover 住这些不同。
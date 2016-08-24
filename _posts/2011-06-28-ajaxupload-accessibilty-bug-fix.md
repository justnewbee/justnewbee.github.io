---
layout: post
title: AjaxUpload Accessibility Bug Fix
date: 2011-06-28 04:06:50
categories: frontend
tags: js
---

好吧，基于之前的 [File Input Accessibility 研究](/2011/06/24/file-input-accessibility.html)，来研究 Accessibility 的 bug 吧。

## 应用场景

![](/images/posts/webex11_button_strip.png)

实际应用中会有 Flash 的介入，但我们目前只研究 Flash 没装或禁用的情况，在这种情况下「Upload」那个按钮里面会插入我的上传控件。

这个菜单是很传统的 `ul>li>a` 的设计，所以每个按钮都是可以通过「Tab」导航，键盘触发的，这就对上传控件提出了两个要求：

1. 不能影响 tab 流
2. 必须能键盘触发，「回车」和「空格」

首先来看看上传控件的结构，其实很简单：

```
<div><input type="file" /></div>
```

file input 很大且透明，保证它的按钮能充满 `div`，并不对用户的视觉造成障碍。

## Fix 1：tab 访问

### 分析

file input 作为一个 input，天生支持「tab」，所以当「tab」到「upload」那个按钮的时候，再次「tab」会进入到 file input，这样就破坏了原有的 tab 顺序。Firefox、Chrome、Safari 需要多按一次「tab」，IE、Opera 则需要多按两次。而 tab 聚焦还会影响屏幕阅读器的阅读，给用户造成混淆。

### 解决

让 file input 不能被正常的「tab」到，设置 `tabIndex` 为 `-1` 即可。测试了一下所有的浏览器都是可以的，但同时对调用者提出了需求：

1. 调用者的节点，必须是 tabbable（具体可以看 jQuery ui core 的 tabbable）

## Fix 2：键盘触发

### 分析

现在「tab」不能进入 file input 了，所以键盘事件不能在 file input 上触发，凡是 tabbable 的元素都支持键盘事件，所以我们可以在调用者节点上调用 file input 的 `click`，但根据之前的分析，只有 IE 和 Safari 可以无障碍地弹 file dialog；Chrome 和 Opera 没有反应；Firefox 则有时会弹一个警告黄条，有时又完全没有反应。

所以对于 IE 和 Safari，我们只需要这样就搞定了。

```js
if ($.browser.msie || $.browser.safari) {
  node.bind("keydown keyup", function(e) {
    switch (e.keyCode) {
    case 13:// ENTER
    case 32:// SPACE
      if (e.type == "keyup") {
        setTimeout(function() {
          $ui.find("input:file").click();
        }, 0);
      }
      return false;// prevent default, SPACE will scroll down page
    default:
      break;
    }
  });
}
```

Firefox / Chrome / Opera 还需要另加处理。根据之前对 _聚焦和键盘操作_ 的分析，在使用代码对 file input 进行聚焦之后按回车和空格，Firefox / Chrome / Opera 都天生响应回车和空格，但由于不能成为「tab」访问的障碍，我们必须做一件事，就是在 file input 上接受「tab」键的时候，必须把焦点正确还给调用者节点的上一个或下一个 tabbable 节点。下一个的话，不需要考虑，浏览器自己的处理就可以了；但是对于上一个，也就是「shift tab」的时候，必须处理，不然会由于之前设置 focus 的事件导致 tab 键被禁锢。

最终的解决方案，对调用者节点：

```js
if (ie || safari) {
  var keyHandler = function(e) {
    switch (e.keyCode) {
    case 13:// ENTER
    case 32:// SPACE
      if (e.type === "keyup") {
        setTimeout(function() {
          $ui.find("input:file").click();
        }, 0);
      }
      return false;// prevent default, SPACE will scroll down page
    default:
      break;
    }
  };
} else {// Firefox / Chrome / Opera only supports triggering file dialog on keyboard events, both SPACE and ENTER, the file input should be focused
  var focusHandler = function(e) {
    setTimeout(function() {
      $ui.find("input:file").focus();
    }, 0);
    return false;
  };
}
```

对 file input：

```js
input.attr("tabIndex", -1);// not to mess up with the tab sequence of node
if (!ie && !safari) {
  input.bind("focus", function(e) {// Firefox/Chrome/Opera we have to focus the file input to enable keyboard access
    e.stopPropagation();// prevent infinate loop
  }).bind("keydown", function(e) {// However we have to handle tab to make the tab sequence the normal way
    // Tab works fine, shift-tab is what we have to deal with, because by default
    // the foucs switches to node, in which case tab key will be trapped.
    if (e.keyCode == 9 && e.shiftKey) {
      setTimeout(function() {
        options.prevTabbable(node).focus();
      }, 0);
      return false;
    }
  });
}
```

## 不可逾越的问题 Known Issues

* IE / Safari 完美越狱
* Firefox / Chrome 在「tab」的时候，会丢失 focus 样式
* Opera 的 keyboard navigation 也可以，但由于它上面的 tabbing focus 样式没显示，所以测试得不太爽利

---
layout: post
title: ie9 inline-block 的文字修改后重叠的 bug
date: 2012-06-21 05:06:34
tags: ie, javascript
---

最近有个 bug，只在 ie9 下才有，ie678 几个却还是好的。

最后发现是跟 `inline-block` 元素有关，凡是像 `text-content <inlineblock element> text-content` 这样的结构，在改变中间元素的内容的时候，这个 bug 就产生了。

## HTML

```html
<div>text before inline-blcok <span id="fuck">wwwwwwwwwwwwwwwwww</span> text after inline block;</div>
<div><input id="suck" type="text" /><button onclick="go();">go</button></div>
```

## CSS

```css
#fuck {
  display: inline-block;
  color: red;
}
```

## JS

```js
function go() {
  var fuck = document.getElementById("fuck"),
    suck = document.getElementById("suck");

  fuck.innerHTML = suck.value;
}
```

将文字由长变短，由短变长，都会产生问题：

![](/images/posts/ie9_display_inlineblcok_layout_issue.png)

解决的办法，就是想办法触发 ie 的 reflow，但是，还必须使用 `setTimeout` 让当前的 `click` 事件先完成。

## JS

```js
function go() {
  var fuck = document.getElementById("fuck"),
    suck = document.getElementById("suck");

  fuck.innerHTML = suck.value;
  fuck.style.display = "inline";
  setTimeout(function() {
    fuck.style.display = "inline-block";
  }, 0);
}
```

当然，可以用 `line-height height` 之类的其他属性，但是 那需要记住原来的值，麻烦点。
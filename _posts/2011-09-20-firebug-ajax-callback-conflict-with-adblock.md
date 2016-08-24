---
layout: post
title: firebug 中 ajax 回调方法里的错误无法显示（与 adblock 冲突）
date: 2011-09-20 05:09:17
categories: software
tags: firebug
---

最近想写一个测试 UI 的小工具，里面会用到很多 AJAX 请求。写的过程中发现，界面并没有如预想一样展现，但是又没有任何的错误。然而，换作 Chrome 或者 Safari，却能在 dev tool 里看到有明显的错误。

这个问题一直困扰了我很久...

一开始我以为是 jQuery 的问题，但是没道理，在别的调试工具里面是好的。所以我又试了 mootools 的方式调用 AJAX，结果一样。最后，来了招狠的，用原生方式调用 AJAX，结果还是一样，代码如下：

```js
var xhr = new XMLHttpRequest();
xhr.open("GET", "index.html", true);
xhr.onreadystatechange = function () {
  console.info(xhr.readyState);
  console.error(kcuf);// SHOULD throw exception and show error in console panel
};
xhr.send(null);
```

所以，只能是 firebug 的问题。正想给 firebug 提 bug，想到之前碰到的插件冲突的问题，想会不会是我的机器上特有的（我的 firefox 插件确实装挺多的）。于是去了别人的机器上试，他们是可以的...

基本上能确认是插件冲突的问题了，先禁用四五个看看，如果再试的结果是好的，就说明罪魁在这几个中间。运气很好，第一批就中了；好运气继续，第二次马上找到的元凶 - adblock plus，因为它的名字最靠前...

去 firebug 上提了个 [bug](http://code.google.com/p/fbug/issues/detail?id=4847)，虽然不是他们的错，但还是要他们来解决的。

## 更新 2011-09-22

今天收到了 firebug 的回信：

> 1. Create a new profile
> 2. Install the latest version of Adblock Plus (currently 1.3.9) from <a href="http://adblockplus.org/en/" target="_blank">http://adblockplus.org/en/</a>
> 3. Go to this issue's page
> 4. Open Firefox's Web Console by pressing Ctrl+Shift+K
> 5. Open Firefox's Scratchpad by pressing Shift+F4
> 6. Enter the code:
> var xhr = new XMLHttpRequest();
> xhr.open("GET", "index.html", true);// use a good url please
> xhr.onreadystatechange = function () {
> console.info(xhr.readyState);
> console.error(kcuf);// should throw exception and show in console
> };
> xhr.send(null);
> 7. Press Ctrl+R

说的很清楚，我也照它上面说的试验了，果然就是 Adblock Plus 的问题，本打算给 Adblock 也报个 bug，不过碰到了 403 错误。也罢，反正我也不是它的粉丝，卸掉就行了。

很开心，还学会了一招，就是 `Ctrl+Shift+K / Shift + F4` 几个快捷键。
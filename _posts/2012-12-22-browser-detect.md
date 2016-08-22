---
layout: post
title: 浏览器侦测browser detect
date: 2012-12-22 09:12:51
tags: javascript, jquery
---

最近的项目，很多人写了很多东西，我的任务是重构整理。

似乎很多人的通病，喜欢自己实现胜过使用既有的东西（很多时候我也这样），或者这么说，没有扫描是否有既有事物的习惯。

就拿浏览器侦测来说，这个项目里面就有两处实现，外加 jQuery 的，共有三个。不过 jQuery 也是，`$.browser` 实在有点弱，怪不得需要有另外的实现。

我所做的事情是，把功能相同但实现不同的代码整理起来，加以优化和强化，并在我们的项目里面使用统一的接口。

这里给出另外写的一个测试代码：

我们需要：

* 当前平台 - `platform` 一般就是操作系统 os
* 浏览器名 - `browser`
* 浏览器版本 - `browserVersion`
* 浏览器版本（数字） - `browserVersionNumber` 用以比较，因为字符串比较很可能产生 `"17" < "7"` 的结果
* 浏览器引擎名 - `browserEngine` 引擎的版本号似乎很少用到，所以忽略

```js
var ua = navigator.userAgent.toLowerCase(),
    browserM = ua.match(/(opera|chrome|safari|firefox|msie)[/s]*([d.]+)/),
    engineM = ua.match(/(presto|webkit|gecko|trident)\/([d.]+)/),
    versionM = ua.match(/version\/([d.]+)/),
    platform = navigator.platform.toLowerCase(),
    browser = navigator.appName.toLowerCase(),
    browserVersion = navigator.appVersion.toLowerCase(),
    browserVersionNumber = 0,
    browserEngine = "other";

if (browserM) {
    browser = browserM[1];
    browserVersion = versionM ? versionM[1] : browserM[2];
}
if (browserVersion) {// convert version string to version number, so that "21.12.09" becomes 21.1209 which make sure version compare is right
    var verionNumberM = browserVersion.match(/^(d+).?(.*)/),
        vMain, vSub;
    if (verionNumberM) {
        vMain = verionNumberM[1];
        vSub = verionNumberM[2].replace(/D/g, "");
        vSub = vSub ? "0." + vSub : "0";
    }
    browserVersionNumber = parseInt(vMain, 10) + parseFloat(vSub);
}

if (engineM) {
    browserEngine = engineM[1];
} else if (browser === "msie") {// older IE don't have Trident/xxx
    browserEngine = "trident";
}

alert([
    "ua: ", ua, "nn",
    "platform: ", platform,    "n",
    "browser: ", browser, "n",
    "browserVersion: ", browserVersion, "n",
    "browserVersionNumber: ", browserVersionNumber, "n",
    "browserEngine: ", browserEngine
].join(""));
```

这是几大主流浏览器下的测试结果：

![](/images/posts/browser_detect.png)

至于肥猪流的浏览器，经过测试，最新 Maxthon4 和 360 都属于 chrome。

浏览器厂商对于 `navigator.userAgent` 似乎从来没有统一的意见过，在 <http://www.useragentstring.com> 这个网站上有极全的信息。也是以上代码的主要参考，对于单个浏览器，可以看以下页面：

* <http://www.useragentstring.com/pages/Netscape/>
* <http://www.useragentstring.com/pages/Firefox/>
* <http://www.useragentstring.com/pages/Chrome/>
* <http://www.useragentstring.com/pages/Opera/>
* <http://www.useragentstring.com/pages/Safari/>
* <http://www.useragentstring.com/pages/Internet%20Explorer/>

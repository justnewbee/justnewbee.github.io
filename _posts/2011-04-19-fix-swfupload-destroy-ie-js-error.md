---
layout: post
title: 修复 SWFUpload destroy 时 IE 下不断报 JS Error 的 bug
date: 2011-04-19 09:04:58
categories: bug
tags:  as, js, upload
---

最近遇到了一个棘手的问题，不得不研究一下 SWFUpload 的 AS 代码。

SWFUpload 的 JS 代码早就已经被我改的面目全非了，改 AS 想来也是迟早的事...

Bug 是这样的，先看图：

![](/images/posts/swfupload_destroy_recursive_error.png)

怎样重现这个 bug 呢，也很容易，作者说不能重现...看来他充其量也只能算是个 NB 的 Flash 开发，哈哈。

首先说明这个 bug 是 SWFUpload 组件本身的，不是我改出来的，下面我们就用官方网站上的 demo 来试验。

按以下步骤操作，肯定可以重现，不过要保证 IE 装了 MS Develop Tools。

1. 打开 IE，如果 Develop Tools 开着，关了它
2. 浏览网页 [http://demo.swfupload.org/v220/simpledemo/index.php](http://demo.swfupload.org/v220/simpledemo/index.php) 在这里你可以随意刷新页面
3. 打开 Develop Tools，注意这时你不能再刷新页面
4. 在 Develop Tools 的 Script 面板里输入代码，我们用官方的 destroy 方法：`SWFUpload.instances["SWFUPload_0"].destroy();`
5. 点「Run Script」执行代码

你可以看到代码执行后 console 里打印的是 `true`，说明 `destroy()` 成功。可是稍微停顿之后 悲剧就发生了...

其实不用官方的 `destroy()` 方法，用以下方式也是一样：

```js
var x = document.getElementsByTagName("object")[0];
x.parentNode.removeChild(x);
```

这说明不是 `destroy()` 这个 JS 方法的问题，是 Flash 内部的问题。

我在自己的代码里加个断点，debug 进去之后发现了两段可疑代码，所有的 JS 都找不到这样的代码，猜应该是 Flash 自动生成的。

第一段：

```js
try {
    document.getElementById("swfupload-1303117382646-0").SetReturnValue(__flash__toXML(SWFUpload.instances["swfupload-1303117382646-0"].testExternalInterface()));
} catch (e) {
    document.getElementById("swfupload-1303117382646-0").SetReturnValue("&lt;undefined/&gt;");
}
```

第二段：

```js
function __flash__arrayToXML(obj) {
    var s = "&lt;array&gt;";
    for (var i = 0; i &lt; obj.length; i++) {
        s += "&lt;property id="" + i + ""&gt;" + __flash__toXML(obj[i]) + "&lt;/property&gt;";
    }
    return s + "&lt;/array&gt;";
}
function __flash__argumentsToXML(obj,index) {
    var s = "&lt;arguments&gt;";
    for (var i = index; i &lt; obj.length; i++) {
        s += __flash__toXML(obj[i]);
    }
    return s + "&lt;/arguments&gt;";
}
function __flash__objectToXML(obj) {
    var s = "";
}
function __flash__escapeXML(s) {
    return s.replace(/&amp;/g, "&amp;amp;").replace(/&lt;/g, "&amp;lt;").replace(/&gt;/g, "&amp;gt;").replace(/"/g, "&amp;quot;").replace(/'/g, "&amp;apos;");
}
function __flash__toXML(value) {
    var type = typeof(value);
    if (type == "string") {
        return "&lt;string&gt;" + __flash__escapeXML(value) + "&lt;/string&gt;";
    } else if (type == "undefined") {
        return "&lt;undefined/&gt;";
    } else if (type == "number") {
        return "&lt;number&gt;" + value + "&lt;/number&gt;";
    } else if (value == null) {
        return "&lt;null/&gt;";
    } else if (type == "boolean") {
        return value ? "&lt;true/&gt;" : "&lt;false/&gt;";
    } else if (value instanceof Date) {
        return "&lt;date&gt;" + value.getTime() + "&lt;/date&gt;";
    } else if (value instanceof Array) {
        return __flash__arrayToXML(value);
    } else if (type == "object") {
        return __flash__objectToXML(value);
    } else {
        return "&lt;null/&gt;";
    }
}
function __flash__addCallback(instance, name) {
    instance[name] = function () {
        return eval(instance.CallFunction("&lt;invoke name="" + name + "" returntype="javascript"&gt;" + __flash__argumentsToXML(arguments, 0) + "&lt;/invoke&gt;"));
    }
}
function __flash__removeCallback(instance, name) {
    instance[name] = null;
}
```

因为它们很可能是 Flash 自己生成的，很可能是 Flash 加载后做的，所以我只能用最土但是最有用的办法，通过 `setTimeout` 把这些代码加在自己的代码后面（需要稍稍改一下上面的代码 NB 的 JS 开发都懂的），并给他们加上 `console`，那样我就可以理直气壮的 debug 到了。

经过 debug 之后就发现了以下的事情：

![](/images/posts/swfupload_destroy_bug_errors_in_debug_mode.png)

AS 中有代码：

```as
this.testExternalInterfaceCallback = "SWFUpload.instances["" + this.movieName + ""].testExternalInterface";
```

完美切合了第一个代码片段，而 `this.testExternalInterfaceCallback` 在 SWFUpload 的构造方法里有 Timer 会每隔一秒钟调用。

```as
// Start periodically checking the external interface
var oSelf:SWFUpload = this;
this.restoreExtIntTimer = new Timer(1000, 0);
this.restoreExtIntTimer.addEventListener(TimerEvent.TIMER, function():void {
    oSelf.CheckExternalInterface();
});
this.restoreExtIntTimer.start();
private function CheckExternalInterface():void {
    if (!ExternalCall.Bool(this.testExternalInterfaceCallback)) {
        this.SetupExternalInterface();
        this.debug("ExternalInterface reinitialized");
        if (!this.hasCalledFlashReady) {
            ExternalCall.Simple(this.flashReadyCallback);
            this.hasCalledFlashReady = true;
        }
    }
}
```

`testExternalInterfaceCallback` 在失败的情况下会认为是 `false`，故而触发 `SetupExternalInterface` 方法，`SetupExternalInterface` 中每一次的 `addCallback` 都会触发一次 `__flash__addCallback` 而产生一个「Object required」错误。

这样上面都走通了，而 Firefox/Safari/Opera/Chorme 下面不会有这样的问题，猜（我只能猜）是它们的回收做的比较好 - 或者 pluin 版本的 FlashPlayer 比 ActiveX 版本的做的好，在 Flash 从 DOM 上移出之后同样把它内部的东西都给销毁了。

接下来就是 Fix 了，很简单，我们只要在执行 JS 的 `destroy` 的时候告诉 Flash 先把 Timer 停掉就行了。

AS 代码，添加方法：

```as
private function Destroy():void {
    if (!this.restoreExtIntTimer) {
        return;
    }
    this.restoreExtIntTimer.stop();
    this.restoreExtIntTimer = null;
}
```

并在 `SetupExternalInterface` 中把它加进去：

```as
ExternalInterface.addCallback("Destroy", this.Destroy);
```

JS 中在 `destroy()` 方法里，适当的地方调用一下：

```js
this.callFlash("Destroy");
```

这样就搞定这个 bug 了。
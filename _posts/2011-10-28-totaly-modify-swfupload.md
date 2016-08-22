---
layout: post
title: 彻底重写 SWFUpload
date: 2011-10-28 05:10:09
tags: actionscript, swfupload
---

项目的需要，用了开源的 SWFUpload。一开始也只是用用，但是后来发现光是用用还不行，必须改它的源代码，才能适合项目的需求。一开始也就改改 JS，通过 jQuery 减小了不少的代码量。可那还不行，有好几个 bug 还必须改 flash 文件。于是造就了 newbie 且 newbee 的 flash 开发的我。

其实之前也就在 AS 里面小打小闹，大的结构也不敢改它。这次，有个新需求过来。我心里早就有的想法蠢蠢欲动，必须趁这个契机，以我的代码风格，把 SWFUpload 改个脱胎换骨。

这两天就在搞这个事情，我的想法是：

1. 让代码风格完全变成我的
2. 去掉基本上没用的
3. 为了保证目前的先用着，新写一个 `FlashUpload`（以下 SWFUpload 是老的，FlashUpload 是新的）

JS 的修改除了代码风格的变化之外，其余的都是为了适应 AS 的变动，在这里记录一下AS代码一些较明显的变化。

## UI

### SWFUpload 方式

`SWFUpload` 希望在 Flash 中提供 UI 的支持，且将默认的 `window mode` 设成 `window`，并提供一大堆的 UI 配置参数：`button_image_url, button_width, button_height, button_text, button_text_style, button_text_top_padding, button_text_left_padding`。

**SWFUpload 批斗**

* 稍微复杂一些的样式，想要通过参数很难或者要额外做一张符合 SWFUpload 要求的背景图片
* 这一大堆参数，不仅给调用者造成困扰，同样也增加了 flash 文件及内存的大小
* `button` 这个词不仅使我产生困惑，我相信作者自己也搞不清了，很明显的一点，他把点击的 handler 命名成 `ButtonClickHandler`，但这个 handler 明明只是绑在 `stage` 上的...

**分析**

在实际的应用过程中，我们都会想着把页面上既有的某个 HTML 节点「改造」成一个上传按钮，不管是把 flash 安插在里面也好，还是鼠标移上去的时候再**覆盖**在上面也好。总之要求 flash 是透明的，而且也不期望你能给我显示个什么文字或图像，不需要。

这么些个参数，导致简单的改变大小都需要调用 flash 的接口，其实如果把 flash 放到一个 div 中，flash完全占满它，就只需要调整 div 的大小即可。因为 flash 是透明的话，不管如何调整，都不会有任何的感觉。实验证明，调整祖先节点的大小，以及 `absolute` 样式的 `top/left` 属性，不会给 flash 产生困扰。但是，如果把 flash 或其祖先节点隐藏的话，在除了 IE 之外的浏览器中都会造成上传中断且无 error 消息...

### FlashUpload 改进

去掉所有关于 button 的设定，`FlashUpload.as` 中不再创建 Button 相关的元素；去掉所有对 JS 的调整 Button 样式的接口；不可尽数，反正又去掉了一大堆的废代码。

## button_action, button_window_mode, button_disabled, button_cursor

### SWFUpload 方式

依然是 button...，但是，我们已经不存在 button 了。经过之前的改动，UI 上只剩下一个全屏的 cursorSprite 用于设置鼠标。

**SWFUpload 批斗**

`button` 这个词实在太容易把人带入误区。

**分析**

我们的目标是 - 没有 `button`。于是这些个词都改了 - action, window_mode, disabled, cursor。

* `disabled` 最好理解，跟原来的作用基本一样，就是控制鼠标点击的时候，让它什么事情都不用做。原来的 `disabled` 还会管 button 背景图的坐标之类的，现在不用了，只是纯粹的一个 flag
* `window_mode` 不需要了，因为我们只求透明
* `cursor` 是负数 int 类型，可取值只有两个，分别代表箭头和手形，默认是手形；像这样的参数，其实用 `boolean` 型对于用户来说是最开心的，而且由于大多数情况下，要求可点击的元素呈手形，我把它改成了 `arrowCursor:Boolean=false`
* `action` 最后来看这个 `action`。`SWFUpload` 提供了四个 `Number` 类型的选项 `SELECT_FILES`（默认）、`SELECT_FILE`、`START_UPLOAD`、`JAVASCRIPT`。对于 `START_UPLOAD` 我实在很难理解，那会是什么样的一个行为呢...`JAVASCRIPT` 估计也极少用到。所以又剩下两个 - 把 `action` 改成 `multiple:Boolean=true` 对于用户来说应该更明朗一些。

### FlashUpload 改进

```
button_action:Number=SELECT_FILES --> multiple:Boolean=true
button_window_mode:String="window" --> 不要了
button_disabled:Boolean=false --> disabled:Boolean=false
button_cursor:Number=ARROW --> arrowCursor:Boolean=false
```

以上改动，同时删除各自对应 JS 及 AS 的静态属性，彻底摆脱 button 这个迷幻蘑菇，又减小了文件及内存。

## ReturnUploadStart

### SWFUpload 方式

`SWFUpload` 中对 `startUpload` 的设计很是奇怪，它是这样一个流程：`JS_startUpload --> AS_StartUpload --> JS_uploadStart --> JS_upload_start_handler --> JS_return_upload_start_handler -(true/false)-> AS_ReturnUploadStart`

它是在 AS_ReturnUploadStart 里面才开始真正进行上传，其中 `AS_ReturnUploadStart` 接受一个 `Boolean` 参数，为 `false` 的话，不会上传文件，并 requeue 它。

**SWFUpload 批斗**

在我看来当 `JS_uploadStart` 的时候，就是 AS 告诉 JS 上传已经开始了，要做任何 validataion 没有必要在调用 `StartUpload` 之后，完全可以在之前就做掉。

我做了一个实验，在 JS 的 `uploadStart` 中返回 `false`，并试图在文件上传完成（失败或者成功）之后上传下一个文件，很容易就造成了一个无限循环。

**分析**

`ReturnUploadStart` 和 `Requeue` 其实是两个几乎用不到的东西，而且 `ReturnUploadStart` 在 JS 中没有对外的接口（JS 端会在设置里面加入自己的 handler - 其实这不安全）。

### FlashUpload 改进

流程简化：`JS_startUpload --> AS_StartUpload --> JS_uploadStart --> JS_upload_start_handler`。

AS 中把 `ReturnUploadStart` 里面的动作放到 `StartUpload` 里面，同时解决了 `MISSING_UPLOAD_URL` 错误的时候，文件状态不是 `error` 而是 `queued` 的问题（它会requeue）。

## file_upload_limit 参数及统计信息

### SWFUpload 方式

`SWFUpload` 提供了文件大小 `file_size_limit`，上传队列大小 `file_queue_limit`，上传成功数的限制 `file_upload_limit`，并提供了取/设统计信息的两个方法 `GetStats` 和 `SetStats`。

**SWFUpload 批斗**

其中 file_upload_limit 最没用，用户刷新了之后照样能在上传，这个参数的存在还增加了 AS 代码的复杂度。有 `GetStats` 无可厚非，但是 `SetStats` 就可笑了，就跟政府机关做报告似的还能作假...

为了这个统计信息，AS 设置了几个属性专门做记录，同样增加了 AS 代码的复杂度、可读性及维护成本。

### FlashUpload 改进

* 去掉 `file_upload_limit`
* 去掉 `Get/SetStats` 两个接口，并去掉那些到处扎根，妨碍代码干净度的统计用属性

## AS 中的 Handler

### SWFUpload 方式

`SWFUpload` 的作者明显是个方法首字母大写派+后缀派，`XxxHandler`。

**SWFUpload 批斗**

我偏要说这样看不清楚，必须看到方法的最后才能知道它是个 Handler，不够一目了然。

**分析**

我是个方法首字母小写派+前缀派。

### FlashUpload 改进

所有的内部事件处理方法都改成 `handleXxx`，并挪在一块，大家都是一样的，挤在一起不会吵架。

## AS 对 JS 调用方式的变化

### SWFUpload 方式

在 `SWFUpload.as` 中用一堆的 `this.xxx_Callback = "SWFUpload.instances["\" + this.movieName + "\"].xxx` 保存所有的对 JS 接口，在 `ExternalCall.as` 定义通用的（`Simple/Bool`）及针对性的（`Debug`、`FileQueued`、`UploadStart`...）等接口。

在 `SWFUpload.as` 中调用 `ExternalCall.as` 中定义的接口完成对 JS 的调用。

**SWFUpload 批斗**

* `this.xxx_Callback` 完全绑死 JS 与 AS，要保证 SWFUpload 有个静态属性 `instances`（这个没有必要），要保证 `instance` 有这些方法（这个确实需要）
* `this.xxx_Callback` 不仅增加了编译后 flash 文件的大小，且增大运行时的内存消耗
* `ExternalCall.as` 中 `ExternalCall.xxx` 的定义是 `ExternalInterface.call(xxx_Callback, EscapeMessage(arg0), ...);` 并没有提到任何实质性的作用

**分析**

`ExternalCall.as` 跟大多数的公务员应该是一样的，吃干饭的家伙就应该下岗；而 `this.xxx_Callbak` 则是 `ExternalCall.as` 吃干饭的工具。必须干掉这两个蛀虫，既减小文件大小，又降低内存消耗。

### FlashUpload 改进


干掉所有的 `this.xxx_Callbak`，端掉整个 `ExternalCall.as`；采用静态 AS 方法调用静态 JS 方法的形式，入口单一，好维护：

```as
// FlashUpload.as
private static function externalCall(movieName:String, fn:String, ...parameters):* {
    switch (parameters.length) {
    case 3:
        return ExternalInterface.call("FlashUpload.callByAS", movieName, fn, escapeMessage(parameters[0]), escapeMessage(parameters[1]), escapeMessage(parameters[2]));
    case 2:
        return ExternalInterface.call("FlashUpload.callByAS", movieName, fn, escapeMessage(parameters[0]), escapeMessage(parameters[1]));
    case 1:
        return ExternalInterface.call("FlashUpload.callByAS", movieName, fn, escapeMessage(parameters[0]));
    default:
        return ExternalInterface.call("FlashUpload.callByAS", movieName, fn);
    }
}

// FlashUpload.js
callByAS: function() {
    var args = Array.prototype.slice.call(arguments, 0),
        movieName = args.shift(),
        fn = args.shift(),
        flashupload = FlashUpload._instances[movieName];

    if (flashupload) {
        return flashupload[fn].apply(flashupload, args);
    }
}
```

## JS 对 AS 调用方式的变化

### SWFUpload 方式

`SWFUpload.as` 中有一个近 50 行的方法用于创建对 JS 可用的接口。

**SWFUpload 批斗**

这 50 行代码不断重复类似的操作，而且是这样的操作：

```as
ExternalInterface.addCallback("xxx", this.xxx);
```

**分析**

其实我们只需要找到所有的 `xxx`，循环一下即可。

### FlashUpload 改进

```as
// FlashUpload.as
private static const EXTERNAL_INTERFACES:Array = ["Destroy", "TestExternalInterface",
    "Browse", "StartUpload", "CancelUpload",
    "GetFile", "AddFileParam", "RemoveFileParam",
    "SetUploadUrl", "SetPostParams", "SetFileTypes", "SetFileSizeLimit", "SetFileQueueLimit", "SetFilePostName",
    "SetUseQueryString", "SetHTTPSuccess", "SetAssumeSuccessTimeout",
    "SetDebug", "SetDisabled", "SetMultiple", "SetArrowCursor"];
private function setupExternalInterface():void {
    for (var i:Number = 0; i &lt; FlashUpload.EXTERNAL_INTERFACES.length; i++) {
        var callback:String = FlashUpload.EXTERNAL_INTERFACES[i];
        try {
            ExternalInterface.addCallback(callback, this[callback]);
        } catch (ex:Error) {
            this.log("Callback "" + callback + "" not set: " + ex.message);
            return;
        }
    }

    externalCall(this.movieName, "cleanup");
}
```

我把所有的方法，都改成了小写字母开头，唯独这些对 JS 开放的接口，把他们弄的特殊点比较好。

## 其他附带的改动

* 去掉参数 `preserve_relative_urls`、`requeue_on_error`
* `SelectFile` 及 `SelectFiles` 合并成一个叫 `Browse` 的接口
* 去掉 `StopUpload`、`RequeueUpload`

其他的就不再赘述，不知道原作者如果看到它的东西被我辣手摧花会是什么个感觉呢...

所以 不要让我认真起来！
---
layout: post
title: AjaxUpload Accessibility 研究（续） - IE 下 Access Denied
date: 2011-08-08 05:08:25
categories: 倒腾
tags: ajaxupload, javascript
---

前面对 file input 的页面行为作了研究，基本上能解决键盘操作的问题。还剩下一个 IE（我的是 9 够新了）下的问题。

当使用 form 提交的时候报 `Access is denied` - 键盘操作的情况下。这是个很严重的问题了，它将导致 IE 下：

1. Accessibility 上的缺陷，用户不能使用键盘操作
2. ActiveX 禁止的时候，不能修复不透明的 bug（如果 `Access is denied` 不存在，则可以把 file 完全隐藏，不用透明度

`AjaxUpload` 已经给我写的比较复杂了，故此用最原初的形式来研究。

## 一个简单的 HTML 页面 内容如下


```html
<input type="file" id="file" name="file" />
<button id="browse">browse and start upload</button>
```

## 添加 `window.onload` 方法

```js
window.onload = function() {
  var browse = document.getElementById("browse"),
    file = document.getElementById("file");

  file.onchange = function() {
    var form = document.createElement("form"),
      input = document.createElement("input");

    document.body.appendChild(form);

    form.action = "/upload.do";
    form.method = "post";
    form.enctype = "multipart/form-data";

    input.type = "hidden";
    input.name = "filename";
    input.value = file.value.substring(file.value.lastIndexOf("\"") + 1);

    form.appendChild(file);
    form.appendChild(input);
    form.submit();
  };

  browse.onclick = function() {
    file.click();
  };
};
```

上面的代码，会使用「软点击」的方式，弹出 file dialog，在选择了文件之后动态的创建 form，并把 file input 放进 form 中，然后提交 form 进行上传操作。结果就报了 `Access is denied`，通过 `console` 或者 `setTimeout` 很容易看出一个问题：

![](/images/posts/fileinput_softclickupload_before_after.png)

移动了之后的 file 没有 value！而这是什么造成的呢，移动？提交？给 `form.submit` 加了个延时，发现是在 `form.submit` 的时候被清空的。这会不会成为一个不可 fix 的问题呢？

上面的分析，至少说明了一点，跟 form 提交到 iframe 没关系。为了研究 file input 的清空问题，把 form 的 target 设成一个 iframe。发现「硬点击」能正确提交，并且不会被清空。到此，基本上可以认为是提交造成的「Access is denied」造成 file input 的清空。

这里要证明的是，把 file input 放到一个动态生成的 form 不会对结果产生影响。所以改 HTML 结构如下：

```html
<form id="form-upload" action="/newbee/upload.do" method="post" enctype="multipart/form-data" target="iframe-hidden">
  <input type="hidden" id="filename" name="filename" />
  <input type="file" id="file" name="file" />
</form>
<button id="browse">browse and start upload</button>
```

改初始化的 JS

```js
window.onload = function() {
  var browse = document.getElementById("browse"),
    file = document.getElementById("file");

  file.onchange = function() {
    document.createElement("filename").value = file.value.substring(file.value.lastIndexOf("\") + 1);
    document.getElementById("form-upload").submit();
  };

  browse.onclick = function() {
    file.click(); 
  };
};
```

结果还是会产生「Access is denied」...看来 IE 下这个问题是无解了。回头看了一下 `AjaxUpload` 的源码，其实以前就看到这几行注释，不过没注意：

```js
// IE will later display 'access denied' error
// if you use using self._input.click()
// other browsers just ignore click()
```

再注意一个细节，虽然在 file input 的可见输入框里面的值可能是 `D:xxxyyyzzzxyz.txt`，但是用 JS 取 value 的时候，其值却是 `C:fakepathxyz.txt`。

看来真的无解？！
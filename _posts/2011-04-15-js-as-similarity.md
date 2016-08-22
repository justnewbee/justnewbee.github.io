---
layout: post
title: JS 和 AS 还真是很有一样的地方
date: 2011-04-15 11:04:56
tags:  develop
---

既然做了 NB 的 Flash 开发员，有必要多做一些记录。

JS 的代码规范我十分讲究，所以从来不会写 `xx == ""` 或 `xx == false` 之类的代码也不会写像下面这样的代码：
```
if (xx == "") {
  xx = "dflt";
}
```

取而代之的是：
```xx = xx || "dflt";```

首先我试了后一种果然一样。

然后我看到 AS 里面诸多 `xx != null` 的代码感觉浑身不爽，所以做了以下测试：

```as
trace(""" == false: " + ("" == false));
trace("" " == false: " + (" " == false));
trace("0 == false: " + (0 == false));
trace("null == undefined: " + (null == undefined));
trace("null == false: " + (null == false));
trace("undefined == false: " + (undefined == false));
```

果如我所料打印结果为：

```
"" == false: true
" " == false: true
0 == false: true
null == undefined: true
null == false: false
undefined == false: false
```

[FlashDevelop](http://www.flashdevelop.org/) 有一点做的挺好：在编译的时候，对于这样的代码有警告「(1374): col: 35 Warning: Variables of type Null cannot be undefined. The value undefined will be type coerced to Null before comparison.」
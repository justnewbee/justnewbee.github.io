在使用HTML5的`required`属性时，碰到了IE6-9的兼容性问题，这里分享一下。

# 问题

具体的细节这里就不说了，最终的问题是使用KISSY的`prop("required")`无法得到想要的布尔值。

# 分析

获取`required`值有以下几种方式：

1. KISSY/jQuery`$node.prop("required")` “最佳”实践  _期望_得到`true/false`值
2. KISSY/jQuery`$node.attr("required")` 但只能得到`String`
3. 直接`node.required` 逼不得已的做法
4. 正则匹配`/\brequired\b/i.test(node.outerHTML)` 最最逼不得已的做法

在JsBin上写了一个bin用来测试：<http://jsbin.com/rayuqe>。

以下是bin里用到的单选按钮，测试脚本会在每个按钮后面输出它的`outerHTML`，`prop("required")`，`attr("required")`和`.required`：

```html
<!-- 这里前三种required是正确的写法
后面三个true/fals/fuck都是不正确的 但浏览器照样认为那是required -->
<input type="radio" />
<input type="radio" required />
<input type="radio" required="" />
<input type="radio" required="required" />
<input type="radio" required="true" />
<input type="radio" required="false" />
<input type="radio" required="fuck" />
```

# 现象

以下是IE不同版本下测试结果（该结论在Win10虚拟机的IE11中得到）：

## IE11/10

这两个版本的标准化已经相当不错了，可以愉快地一起玩耍了，使用`prop("required")`可以获得正确的布尔值。

![IE 11](/images/required_in_ie11.jpg)
![IE 10](/images/required_in_ie10.jpg)

## IE9

不是我什么都不想要，而是她什么都不想给。这里有一点可以看到，jQuery的`attr()`方法强过KISSY。但如果用的是KISSY，只能降级到最最不想用的正则匹配了。

![IE 9](/images/required_in_ie9.jpg)

## IE 8/7

`prop("require")`返回的是字符串类型，KISSY的`attr("require")`可能返回`undefined`，所以如果要正确地获得`required`为布尔值的话，可以采用`!!propRequired || propRequired === ""`。

![IE 8](/images/required_in_ie8.jpg)
![IE 7](/images/required_in_ie7.jpg)

# 解决问题

总结起来，要获取某`input`的`required`值，需要被逼无奈地使用UA检测，以下是使用KISSY的代码片段：

```
...
if (UA.ie && UA.ie === 9) {
	return /\brequired\b/i.test(input.outerHTML);
}

var required = $input.prop("required");
return !!required || required === "";// required === "" for IE6-8
```

而使用jQuery的话，可以这样：

```
...
return !!$input.attr("required");
```

# 展望

当然这都不是我们想要的，我们需要的是KISSY和jQuery可以帮我们cover住这些不同。
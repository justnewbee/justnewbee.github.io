---
layout: post
title: JS 类继承
date: 2010-10-10 11:10:54
categories: frontend
tags: js
---

面试前端开发的时候经常会问及类和继承，一般人都会说点上来：

1. 对象冒充
2. `call / apply`
3. 原型链

对象冒充和 `call / apply` 并没区别，我们可以看作是同一个，看看区别：

```js
// super-class A
var A = function(name) {
    this.name = name || "A";
};
A.prototype.fuck = "Fuck";
A.prototype.you = "U";
A.prototype.fuckyou = function() {
    console.info(this.fuck + " " + this.you);
};
// sub-class B using call
var B = function(name) {
    A.call(this, name || "B");
};
// sub-class C using prototype-chain
var C = function(name) {};
C.prototype = new A();
var D = function(name) {
    A.call(this, name || "D");
};
// sub-class D using call and prototype-chain
D.prototype = new A();
var a = new A("AA"),
    b = new B("BB"),
    c = new C("CC"),
    d = new D("DD");
// print results
console.dir(a);
console.dir(b);
console.dir(c);
console.dir(d);
console.info("instanceof:", a instanceof A, b instanceof A, c instanceof A, d instanceof A);
console.info("hasOwnProperty(name):", a.hasOwnProperty("name"), b.hasOwnProperty("name"), c.hasOwnProperty("name"), d.hasOwnProperty("name"));
```

![](/images/posts/js_inherit_diff.png)

结果很明显了：

* `instanceof` 说明了对象冒充只是借尸还魂了一下 并没有真正做到继承
* `hasOwnProperty` 说明了只有在构造器里赋的值才是真正属于自己的

对象冒充继承了构造器里的赋值并可以往构造器里传参数，原型链继承了所有的属性但是不能往构造器里传参，结合起来就完美了。

另外还有个区别，对象冒充的方式可以实现多重继承，原型链则可以实现深度继承。

最后要说的一点是，JS 不必那么复杂，滥用技巧往往是性能和维护性的杀手。
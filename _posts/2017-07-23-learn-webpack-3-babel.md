---
layout: post
title: webpack 学习 3 - babel
date: 2017-07-23 10:50:00
categories: develop
tags: js, webpack
---

这一篇的目标是——

编译 ES6/7 的代码。

---

通过这一篇，我们将了解：

* 什么是 _loader_
* 利用 _babel-loader_ 「编译」ES6/7 的其他特性
* 利用 _es3ify-loader_ 解决 IE8 不认关键字做属性的问题

---

修改版本号：`yarn version --new-version 0.0.3`。

# 引子

在 [第一篇] 中，我们已经了解到，虽然 webpack 原生支持 `import` 和 `export`，但对于其他的 ES6/7 特性却并没有处理，所以现在构建出来的代码并不能在所有的浏览器里都能玩。

正如 webpack [官方文档](https://webpack.js.org/guides/getting-started/#es2015-modules)中说的：

> Note that webpack will not alter any code other than import and export statements. If you are using other [ES2015 features](http://es6-features.org/), make sure to use a transpiler such as [Babel](https://babeljs.io/) or [Bublé](https://buble.surge.sh/guide/). 

我们就 Babel 吧。

# loader（加载器）

先了解一下 [webpack 的 loader（加载器）][loader] 的概念。

简言之，webpack 视一切为模块，但 JS 毕竟只认识 JS，所以 loader 的作用就是帮 webpack 「认识」这些「外乡人」，并知道如果处理它们。

你可以在这里找到有用的 [loader 列表](https://webpack.js.org/loaders/)，其中之一就是 [babel-loader]，这正是我们需要的。

# 安装 `babel-loader`

安装以下依赖：

```bash
yarn add babel-loader babel-core babel-preset-env --dev
```

> 这里除了 `babel-loader` 之外，`babel-core` 和 `babel-preset-env` 也是需要的，否则编译会报错。
> [babel-core](https://github.com/babel/babel/tree/master/packages/babel-core) 是 babel 的编译核心；
> [babel-preset-env](https://github.com/babel/babel-preset-env) 能让我们免于写一堆的 presets。

# 修改配置

然后，我们修改一下 `webpack.config.js`，添加如下代码：

```js
module: {
  rules: [{
    test: /\.js$/,
    exclude: /node_modules/,
    use: {
      loader: "babel-loader",
      options: {
        presets: ["env"]
      }
    }
  }]
}
```

> 注意，这里的 `presets` 仅为 _env_。私下认为这并不是一个很好的设计，因为全局搜索代码将无法搜到 `babel-preset-env` 的引用。

较一开始的配置，我们多了一个 [module](https://webpack.js.org/configuration/module/)，这是我们为各种类型的文件指定 _loader_ 的地方，_loader_ 会帮助 webpack 把各种文件解释成 webpack 能够理解的 JS module。

> 参考文档 https://webpack.js.org/concepts/#loaders

这里，我们告诉 webpack：「项目下以 `.js` 结尾的文件（忽略 `node_modules` 目录），请用 `babel-loader` 进行处理，`babel-loader` 的配置也给你了，用那个叫做 `env` 的 _preset_。」

# 构建结果

再次运行 `yarn build`，再去看构建出来的代码：

```js
/******/ (function(modules) {
/******/ // webpackBootstrap
/******/ })
/******/ ([
/* 0 */
/***/ (function(module, exports, __webpack_require__) {
"use strict";

var _component = __webpack_require__(1);
var _component2 = _interopRequireDefault(_component);

function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { default: obj }; }

document.body.appendChild((0, _component2.default)());
/***/ }),
/* 1 */
/***/ (function(module, exports, __webpack_require__) {
"use strict";

Object.defineProperty(exports, "__esModule", {
  value: true
});

exports.default = function () {
  var text = arguments.length > 0 && arguments[0] !== undefined ? arguments[0] : "hello webpack";
  var element = document.createElement("div");

  element.innerHTML = text;

  return element;
};
/***/ })
/******/ ]);
```

之前留存的 ES6 代码（箭头函数、参数默认值、`const`）都已经被转成所有浏览器都能认识的了。

# IE8

上面的代码中有 `exports.default = ...` 这样的代码，在 IE8 中是会报错的，会说「缺少标识符」。虽然 IE8 比较老，也是我们比较痛恨的浏览器，但可能还是需要解决它。

如果你愿意花时间尝试，对象属性名称为关键字的话，也是没有被加引号的：

```js
console.log({
  break: 'break',
  catch: 'catch',
  default: 'default'
});
```

这些同样在 IE8 下会报错。

我们需要 [es3ify](https://www.npmjs.com/package/es3ify) 来对代码进行转换，对于 webpack 而言，只需要 [es3ify-loader] 就行了。

```bash
yarn add es3ify-loader --dev
```

然后修改 _webpack.config.js_ 的 _module_：

```js
module: {
  rules: [{
    test: /\.js$/,
    exclude: /node_modules/,
    use: ["es3ify-loader", {
      loader: "babel-loader",
      options: {
        presets: ["env"]
      }
    }]
  }]
}
```

> 注意，_es3ify-loader_ 必须放在 _babel-loader_ 前面，否则会报错说「Illegal import declaration」。

这里，你应该已经注意到了 _use_ 的多种写法，它可以支持字符串，对象，已经字符串和对象的混合数组。

执行构建 `yarn build`，然后再看 _dist/index.js_ 就会发现原来的 `.default` 变成了 `["default"]`。

# 总结

这一节，我们：

1. 首次接触了 webpack 的 [loader] 概念
2. 使用 [babel-loader] 及它的依赖完成 ES6/7 到 ES5 的编译
3. 使用 [es3ify-loader]，兼容了 IE8 下关键字不可做属性名的问题

同样，打个 tag：

```bash
git add .
git commit -m 'CHORE use babel-loader and fix ie8 keyword issue'
git push
git tag 0.0.3
git push origin 0.0.3
```

代码参考：<https://github.com/justnewbee/learn-webpack/tree/0.0.3>

**打完收工**

[loader]: https://webpack.js.org/concepts/loaders/
[babel-loader]: https://webpack.js.org/loaders/babel-loader/
[es3ify-loader]: https://www.npmjs.com/package/es3ify-loader
---
layout: post
title: webpack 学习 5 - TypeScript
date: 2017-08-03 0:27:07
categories: develop
tags: js, webpack
---

这一篇的目标是——

用 [TypeScript] 来写 JS 代码。

---

通过这一篇，我们将了解：

* 如何让 webpack 正确处理 `.ts` 文件
* webpack 配置中 `resolve` 的作用

---

修改版本号：`yarn version --new-version 0.0.5`。

# 简单粗暴，改后缀

很好，CSS 有它的超集 LESS，用起来很爽利。虽然 ES6 已经很好用了，但是我还想要更超的超集呢？换言之，我想用 [TypeScript]。

> TypeScript is a typed superset of JavaScript that compiles to plain JavaScript.

既然是一个「超集」，那么就表示对现有的代码，只需要改一个后缀名就行了。那么我们把组件 _h1_ 改一下：

```bash
git mv src/component/h1/index.js src/component/h1/index.ts
```

执行构建，结果：

```bash
Hash: cbead3971cd2f050057d
Version: webpack 3.3.0
Time: 1162ms
   Asset     Size  Chunks             Chunk Names
index.js  2.87 kB       0  [emitted]  main
   [0] ./src/index.js 240 bytes {0} [built]

ERROR in ./src/index.js
Module not found: Error: Can't resolve './component/h1/' in '/Users/jianchunwang/Workspaces/github-me/learn-webpack/src'
 @ ./src/index.js 3:9-35
error Command failed with exit code 2.
```

找不到了！

那改改 _src/index.js_ 吧：

```js
import h1 from "./component/h1/index.ts";

document.body.appendChild(h1());
```

然后再执行构建，结果：

```bash
Hash: 3d26041417dc2f63d9d1
Version: webpack 3.3.0
Time: 2937ms
    Asset       Size  Chunks             Chunk Names
 index.js    3.48 kB       0  [emitted]  main
index.css  271 bytes       0  [emitted]  main
   [0] ./src/index.js 264 bytes {0} [built]
   [1] ./src/component/h1/index.ts 199 bytes {0} [built]
   [2] ./src/component/h1/index.less 41 bytes {0} [built]
   [3] ./node_modules/css-loader!./node_modules/postcss-loader/lib!./node_modules/less-loader/dist/cjs.js!./src/component/h1/index.less 449 bytes [built]
    + 3 hidden modules
Child extract-text-webpack-plugin /Users/jianchunwang/Workspaces/github-me/learn-webpack/node_modules/extract-text-webpack-plugin/dist /Users/jianchunwang/Workspaces/github-me/learn-webpack/node_modules/css-loader/index.js!/Users/jianchunwang/Workspaces/github-me/learn-webpack/node_modules/postcss-loader/lib/index.js!/Users/jianchunwang/Workspaces/github-me/learn-webpack/node_modules/less-loader/dist/cjs.js!/Users/jianchunwang/Workspaces/github-me/learn-webpack/src/component/h1/index.less:
       [0] ./node_modules/css-loader!./node_modules/postcss-loader/lib!./node_modules/less-loader/dist/cjs.js!./src/component/h1/index.less 449 bytes {0} [built]
        + 1 hidden module
Done in 3.84s.
```

貌似没问题？构建成功了？跑一下 _index.html_，也是可以的。

然而…真的可以了吗？我们看一下构建出来的 _dist/index.js_，中相关的部分：

```js
/* 0 */
/***/ (function(module, exports, __webpack_require__) {
"use strict";

var _index = __webpack_require__(1);
var _index2 = _interopRequireDefault(_index);

function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { "default": obj }; }

document.body.appendChild((0, _index2["default"])());
/***/ }),
```

看来，webpack 只是把它当成了一般的 JS 模块了，但因为后缀不是 `.js`，所以并没有进行 Babel 编译，而只是把内容原封不动地抄了过去。

# 加料 - 增加基础的 ts 语法

没有编译成 ES5 这个问题暂且不表，继续简单粗暴，给 _h1/index.ts_ 加一些简单的 ts 特性，这里就只加一下 ts 的[类型声明](https://www.typescriptlang.org/docs/handbook/basic-types.html)：

```ts
import "./index.less";

export default (text : string = "hello webpack") : Element => {
  const element : Element = document.createElement("h1");
  
  element.className = "h1";
  element.innerHTML = text;
  
  return element;
};
```

构建，报错了：

```bash
Hash: 1cbe910b2b87cb8fd960
Version: webpack 3.3.0
Time: 1091ms
   Asset     Size  Chunks             Chunk Names
index.js  3.18 kB       0  [emitted]  main
   [0] ./src/index.js 264 bytes {0} [built]
   [1] ./src/component/h1/index.ts 377 bytes {0} [built] [failed] [1 error]

ERROR in ./src/component/h1/index.ts
Module parse failed: /Users/jianchunwang/Workspaces/github-me/learn-webpack/src/component/h1/index.ts Unexpected token (3:21)
You may need an appropriate loader to handle this file type.
| import "./index.less";
|
| export default (text : string = "hello webpack") : Element => {
|   const element : Element = document.createElement("h1");
|
 @ ./src/index.js 3:13-47
error Command failed with exit code 2.
```

需要一个 loader 了。

所以，这里我们有这么一些问题：

1. 对于 `.ts`，我们必须把后缀名带上？那像 js 那样可以简单写成 `../module/` 的方式，必须写成 `../module/index.ts`，这太麻烦了
2. 绝对需要一个 loader，没有它，即使不加 ts 特性的纯 ES6/7 语法都不能被正确编译

# 让 `.ts` 文件跟 `.js` 一样有地位

...甚至优先级更高。

webpack 只认识 JS，所以对 JS 文件它有这样的处理：

* `path/to/module` 会找 _path/to/_ 下面的 _module.js_
* `path/to/module/` 会找 _path/to/module/_ 下面的 _index.js_

所以，很多情况下，我们会「偷懒」以 `path/to/module/` 代替 `path/to/module/index`（其实是为了保持代码的简洁）。但是，改成 `.ts` 了之后，每个模块都必须携程 `path/to/module.ts` 或 `path/to/module/index.ts` 的话，我是宁死不干的。

那么就…改回 `.js` 去？

别急，webpack 提供了 [resolve.extensions](https://webpack.js.org/configuration/resolve/#resolve-extensions) 配置，可以让你轻松甩掉后缀包袱。修改 _webpack.config.js_，添加：

```
resolve: {
  extensions: [".ts", ".js", ".json"]
}
```

这样，可以保证，万一目录 _path/to/module/_ 下有 `index.ts` 和 `index.js`，`import path/to/module/` 所加载到的是 `path/to/module/index.ts`。

这样，就可以去掉 _src/index.js_ 中的 `index.ts` 这个看起来令人不爽的冗余代码了。不出意外，构建报的错跟之前是一样的，这说明，`resolve.extensions` 生效了。

# 加 loader，真正构建

webpack 关于 loader 的 [官方文档](https://webpack.js.org/loaders/#transpiling) 中说目前有两个 loader 是来处理 ts 的，分别是 [ts-loader] 和 [awesome-typescript-loader][ats-loader]（简称 _ats-loader_）。

两个差不太多，后者要稍微简单一些，毕竟后者可以不用加 `tsconfig.js`。不过鉴于 [ts-loader] 看起来更「官方」一些，这里还是选择了 [ts-loader]。

## 安装和配置

```bash
yarn add ts-loader typescript --dev
```

> 同时 `typescript` 也是需要安装的，否则构建会报错并要求你安装 typescript。
> 就像 'babel-loader'、`less-loader` 一样，这些「transpiler」加载器似乎都需要我们帮它搞。
> [ats-loader] 也一样。

在 _webpack.config.js_ 中的 `module.rules` 数组中加入一条：

```js
{
  test: /\.ts$/,
  loader: "ts-loader"
}
```

此时若尝试构建，会报错：

```bash
ts-loader: Using typescript@2.4.2
Hash: c9d045067ae16eaf0bed
Version: webpack 3.3.0
Time: 1466ms
   Asset     Size  Chunks             Chunk Names
index.js  2.88 kB       0  [emitted]  main
   [0] ./src/index.js 240 bytes {0} [built]
   [1] ./src/component/h1/index.ts 94 bytes {0} [built] [failed] [2 errors]

ERROR in error TS18002: The 'files' list in config file 'tsconfig.json' is empty.
 @ ./src/index.js 3:9-35

ERROR in ./src/component/h1/index.ts
Module build failed: error while parsing tsconfig.json
 @ ./src/index.js 3:9-35
error Command failed with exit code 2.
```

我们需要一个配置文件，建一个吧：

```bash
echo {} > tsconfig.json
```

所以，你看到其实什么内容都没有。里面具体可以写些什么，可以查看 [tsconfig.json 的文档](http://www.typescriptlang.org/docs/handbook/tsconfig-json.html)。

> 觉得各种配置文件越来越多了啊！如果能全部写在 _webpack.config.js_ 中就好了。

```json
ts-loader: Using typescript@2.4.2 and /Users/jianchunwang/Workspaces/github-me/learn-webpack/tsconfig.json
Hash: f07a5d689ef3756100cf
Version: webpack 3.3.0
Time: 4095ms
    Asset       Size  Chunks             Chunk Names
 index.js    3.19 kB       0  [emitted]  main
index.css  357 bytes       0  [emitted]  main
   [0] ./src/index.js 240 bytes {0} [built]
   [1] ./src/component/h1/index.ts 289 bytes {0} [built]
   [2] ./src/component/h1/index.less 41 bytes {0} [built]
   [3] ./node_modules/css-loader!./node_modules/postcss-loader/lib!./node_modules/less-loader/dist/cjs.js!./src/component/h1/index.less 539 bytes [built]
    + 3 hidden modules
Child extract-text-webpack-plugin /Users/jianchunwang/Workspaces/github-me/learn-webpack/node_modules/extract-text-webpack-plugin/dist /Users/jianchunwang/Workspaces/github-me/learn-webpack/node_modules/css-loader/index.js!/Users/jianchunwang/Workspaces/github-me/learn-webpack/node_modules/postcss-loader/lib/index.js!/Users/jianchunwang/Workspaces/github-me/learn-webpack/node_modules/less-loader/dist/cjs.js!/Users/jianchunwang/Workspaces/github-me/learn-webpack/src/component/h1/index.less:
       [0] ./node_modules/css-loader!./node_modules/postcss-loader/lib!./node_modules/less-loader/dist/cjs.js!./src/component/h1/index.less 539 bytes {0} [built]
        + 1 hidden module
Done in 4.96s.
```

这时再去看看 `[1] ./src/component/h1/index.ts` 对应的 _dist/index.js_ 中的结果：

```js
/* 1 */
/***/ (function(module, exports, __webpack_require__) {
"use strict";

exports.__esModule = true;
__webpack_require__(2);
exports["default"] = function (text) {
    if (text === void 0) { text = "hello webpack"; }
    var element = document.createElement("h1");
    element.className = "h1";
    element.innerHTML = text;
    return element;
};
/***/ }),
```

> 这里并没有用 es3ify，然而却天然没有 babel 的关键词陷阱。

# 总结

这一节，我们：

1. 在 _webpack.config.js_ 中增加了 `resolve.extensions` 让 webpack 可以在没有后缀的情况下知道可以去加载 `.ts` 文件
2. 使用 [ts-loader] 进行编译

打个 tag 先：

```bash
git add .
git commit -m 'CHORE ts'
git push
git tag 0.0.5
git push origin 0.0.5
```

代码参考：<https://github.com/justnewbee/learn-webpack/tree/0.0.5>

**打完收工**

[TypeScript]: https://www.typescriptlang.org
[ts-loader]: https://github.com/TypeStrong/ts-loader
[ats-loader]: https://github.com/s-panferov/awesome-typescript-loader
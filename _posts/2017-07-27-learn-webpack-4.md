---
layout: post
title: webpack 学习 4 - CSS
date: 2017-07-27 0:50:00
categories: develop
tags: js, webpack
---

这一篇，我们将了解：

* 如何叫 webpack 知道怎么处理 CSS
* 如何将 CSS 输出成文件
* 用 LESS 代替 CSS
* 用 PostCSS 处理 CSS 兼容性问题

---

目标：用 LESS 写 CSS，无需关心兼容性。

修改版本号：`yarn version --new-version 0.0.4`。

# 调整代码结构

WEB 应用肯定需要 CSS，在加入 CSS之前，我们对原有的文件稍稍做一些调整。

实际应用中，组件（_component_）这个东西会随着项目的复杂度增大而越来越多，较佳实践是把一个模块作为一个目录，目录名为模块名，目录下的 `index.js` 是其入口文件。因为可能会有多个模块，我们作如下更改：

```
mkdir -p src/component/h1
git mv src/component.js src/component/h1/index.js
```

然后我们稍微改一下代码：

**src/component/h1/index.js**

```js
import "./index.css"; // 这里增加对 CSS 的引用

export default (text = "hello webpack") => {
  const element = document.createElement("h1");
  
  element.className = "h1"; // 引用 className
  element.innerHTML = text;
  
  return element;
};
```

**src/component/h1/index.js**

```js
import h1 from "./component/h1/"; // 改一下路径和模块名称

document.body.appendChild(h1());
```

> 此时若执行构建，铁定出错，因为 `index.css` 这个模块并不存在。

# 添加 CSS 文件，并使之可用

然后我们添加一个 CSS 文件：

```bash
touch src/component/h1/index.css
```

加上几行 CSS 代码：

```css
.h1 {
  color: #f00;
}
```

执行 `yarn build` 试试...报错：

```bash
Hash: f41a031af1e8f7ca9150
Version: webpack 3.3.0
Time: 1184ms
   Asset     Size  Chunks             Chunk Names
index.js  3.47 kB       0  [emitted]  main
   [0] ./src/index.js 240 bytes {0} [built]
   [1] ./src/component/h1/index.js 364 bytes {0} [built]
   [2] ./src/component/h1/index.css 239 bytes {0} [built] [failed] [1 error]

ERROR in ./src/component/h1/index.css
Module parse failed: /Users/jianchunwang/Workspaces/github-me/learn-webpack/src/component/h1/index.css Unexpected token (1:0)
You may need an appropriate loader to handle this file type.
| .h1 {
|   color: #f00;
| }
 @ ./src/component/h1/index.js 7:0-22
 @ ./src/index.js
error Command failed with exit code 2.
```

> You may need an appropriate loader to handle this file type.

## css-loader - 让 webpack 能够理解 CSS 文件

前面说了 webpack 只认识 JS 模块，其他类型的文件需要不同的 loader 来帮助。

于是需要加 [css-loader](https://webpack.js.org/loaders/css-loader/)：

```bash
yarn add css-loader --dev
```

然后修改 `webpack.config.js`，在 `module.rules` 中添加一条：

```js
{
  test: /\.css$/,
  use: ["css-loader"]
}
```

执行 `yarn build`：

```bash
Hash: 8edfc81796a38b7fc5cd
Version: webpack 3.3.0
Time: 1555ms
   Asset     Size  Chunks             Chunk Names
index.js  5.72 kB       0  [emitted]  main
   [0] ./src/index.js 240 bytes {0} [built]
   [1] ./src/component/h1/index.js 364 bytes {0} [built]
   [2] ./src/component/h1/index.css 193 bytes {0} [built]
    + 1 hidden module
Done in 2.33s.
```

没有报错，CSS 能被正确处理。然！并没有 CSS 文件输出，打开 `index.html`  发现 `.h1` 也没有生效。

查看 `dist/index.js`，我们看到 CSS 其实已经被识别了（这里代码被我简化了，去了一些注释）。

```js
/* 2 */
(function(module, exports, __webpack_require__) {
  exports = module.exports = __webpack_require__(3)(undefined);
  
  exports.push([module.i, ".h1 {\n\tcolor: #f00;\n}", ""]);
})
```

而且加入了 `css-loader` 注入的代码（应该就是构建输出中说的「+ 1 hidden module」）。

```js
/* 3 */
(function(module, exports) {
/*
	MIT License http://www.opensource.org/licenses/mit-license.php
	Author Tobias Koppers @sokra
*/
// css base code, injected by the css-loader
// ...
})
```

## style-loader - 让 CSS 可以工作

从上面模块 `#2` 的输出看来，只是表明了 webpack 能够解析 CSS 为一个模块，但这个模块并不能够被 HTML 所理解，所以我们还需要一个 [style-loader](https://webpack.js.org/loaders/style-loader/) ，它所做的事情比较简单，就是「Adds CSS to the DOM by injecting a `<style>` tag」。

```bash
yarn add style-loader --dev
```

然后更新 `webpack.config.js` 中关于 CSS 的部分：

```js
{
  test: /\.css$/,
  use: ["style-loader", "css-loader"] // 增加 style-loader
}
```

构建输出：

```bash
Hash: c3097cc2fdf2ded9b496
Version: webpack 3.3.0
Time: 1627ms
   Asset     Size  Chunks             Chunk Names
index.js  18.6 kB       0  [emitted]  main
   [0] ./src/index.js 240 bytes {0} [built]
   [1] ./src/component/h1/index.js 364 bytes {0} [built]
   [2] ./src/component/h1/index.css 1.02 kB {0} [built]
   [3] ./node_modules/css-loader!./src/component/h1/index.css 193 bytes {0} [built]
    + 3 hidden modules
Done in 2.46s.
```

这时我们打开 `index.html` 看到样式已经起作用了，并且起作用的 CSS 代码处于 `head` 的一个 `style` 标签中（如果你愿意试验，你会发现没添加一个 CSS 文件的内容就会生成一个 `style` 元素）。

查看 `dist/index.js`，会找到一个创建 `style` 元素的模块（`#5`）和一些带有 `node_modules` 标识的代码（这样看来，这并不是我们需要的最终结果）。

## 我要 CSS，文件！

虽然样式已经工作了，但是，「样式就该是样式的样儿」——我希望样式可以构建成一个 `index.css` 文件（一个入口 JS 对应一个出口 JS + 一个出口 CSS）。

这次，没有 loader 帮我们了。我们要的是一个叫做  [extract-text-webpack-plugin](https://github.com/webpack-contrib/extract-text-webpack-plugin) 的插件，它用以从 bundle 中抽离文本到独立的文件中。

```bash
yarn add extract-text-webpack-plugin --dev
```

> 我碰到了一个警告 「warning "extract-text-webpack-plugin@2.1.2" has incorrect peer dependency "webpack@^2.2.0".」，不过运行并没有问题。而过了几天，当我再次安装的时候，这个 plugin 已经升级到了 3.0.0，警告也随之不见了。

更新 `webpack.config.js`：

```js
const path = require("path");
const ExtractTextPlugin = require("extract-text-webpack-plugin"); // 引入

module.exports = {
  entry: "./src/index.js",
  output: {
    path: path.resolve(__dirname, "dist"),
    filename: "index.js"
  },
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
    }, {
      test: /\.css$/,
      use: ExtractTextPlugin.extract({ // 原来是 ["style-loader", "css-loader"]
        fallback: "style-loader",
        use: "css-loader"
      })
    }]
  },
  plugins: [ // 增加 Plugin
    new ExtractTextPlugin("index.css"),
  ]
};
```

构建结果：

```bash
Hash: f2c53e4de8c6762a1f0b
Version: webpack 3.3.0
Time: 1645ms
    Asset      Size  Chunks             Chunk Names
 index.js   3.27 kB       0  [emitted]  main
index.css  22 bytes       0  [emitted]  main
   [0] ./src/index.js 240 bytes {0} [built]
   [1] ./src/component/h1/index.js 364 bytes {0} [built]
   [2] ./src/component/h1/index.css 41 bytes {0} [built]
   [3] ./node_modules/css-loader!./src/component/h1/index.css 193 bytes [built]
    + 3 hidden modules
Child extract-text-webpack-plugin /Users/jianchunwang/Workspaces/github-me/learn-webpack/node_modules/extract-text-webpack-plugin/dist /Users/jianchunwang/Workspaces/github-me/learn-webpack/node_modules/css-loader/index.js!/Users/jianchunwang/Workspaces/github-me/learn-webpack/src/component/h1/index.css:
       [0] ./node_modules/css-loader!./src/component/h1/index.css 193 bytes {0} [built]
        + 1 hidden module
Done in 2.54s.
```

> extract-text-webpack-plugin@2.1.2 的时候有个警告「DeprecationWarning: Chunk.modules is deprecated. Use Chunk.getNumberOfModules/mapModules/forEachModule/containsModule instead.」，升级后也消失了。

可以看到 emit 了一个 `index.css`，此时再去看 `dist/index.js`，会发现这个文件干净多了，你会发现模块 `#2` 的内容只剩下下面这些了：

```js
/* 2 */
(function(module, exports) {
// removed by extract-text-webpack-plugin
})
```

这个时候我们打开 `index.html` 会发现样式效果又没了，原因很简单，HTML 中并没有引入 CSS 文件，引一下就没问题了：

```html
<link rel="stylesheet" href="./dist/index.css" />
```

> 这里我们可以看到有些不方便了，HTML 中引入了构建生成的文件，我们需要经常切换来切换去地改。这个问题，后面来解决，现下仍旧继续关注 CSS 的部分。

# 使用 LESS

现在看来已经可以「打完收工」了？

嗯...实际上，我已经好久没有写原始的 CSS 了，这年头不会个 「CSS 超集语言」已经很难混了，可供选择的有 [LESS](http://lesscss.org/)、[SCSS](http://sass-lang.com/) 和 [stylus](http://stylus-lang.com/)。我最熟的是 LESS，所以我们就来搞搞 LESS 吧。

LESS 是 CSS 的一个超集，所以我们只要把之前的 `.css` 文件改成 '.less'，病修改 JS 中的 `import` 语句就行了。这时如果尝试进行构建（`yarn build` 或 `npm run build`）会发现如下输出：

```
Hash: 2c761ba419f892b8cddf
Version: webpack 3.3.0
Time: 1151ms
   Asset     Size  Chunks             Chunk Names
index.js  3.47 kB       0  [emitted]  main
   [0] ./src/index.js 240 bytes {0} [built]
   [1] ./src/component/h1/index.js 365 bytes {0} [built]
   [2] ./src/component/h1/index.less 240 bytes {0} [built] [failed] [1 error]

ERROR in ./src/component/h1/index.less
Module parse failed: /Users/jianchunwang/Workspaces/github-me/learn-webpack/src/component/h1/index.less Unexpected token (1:0)
You may need an appropriate loader to handle this file type.
| .h1 {
|   color: #f00;
| }
 @ ./src/component/h1/index.js 7:0-23
 @ ./src/index.js
error Command failed with exit code 2.
```

又是「You may need an appropriate loader to handle this file type.」。嗯，官网上直接就有 [less-loader](https://webpack.js.org/loaders/less-loader/)。

```bash
yarn add less-loader less --dev
```

> 虽然我们的代码中并没有用到 `less` 这个 npm 包，然！`less-loader` 对它有依赖，而且必须由我们来装，否则会报错说「Cannot find module 'less'」。

修改 _webpack.conf.js_，告诉 webpack 如何处理 _.less_ 和 _.css_ 文件，修改其中 module.rules 中 CSS 配置如下：

```js
{
  test: /\.(css|less)$/,
  use: ExtractTextPlugin.extract({
    use: ["css-loader", "less-loader"],
    fallback: "style-loader" // use style-loader in development
  })
}
```

> 因为 less 是 css 的超集，因此对于 .css 文件，less-loader 也能处理，所以 `test` 中也包含了 css。很多情况下，同一个项目中是有可能出现两种文件同时存在的情况（虽然我十分厌恶这样情况发生）。

执行构建 `yarn build`，构建成功：

```bash
Hash: 775e0540e5bc6b25cc6e
Version: webpack 3.3.0
Time: 1830ms
    Asset      Size  Chunks             Chunk Names
 index.js   3.27 kB       0  [emitted]  main
index.css  23 bytes       0  [emitted]  main
   [0] ./src/index.js 240 bytes {0} [built]
   [1] ./src/component/h1/index.js 365 bytes {0} [built]
   [2] ./src/component/h1/index.less 41 bytes {0} [built]
   [3] ./node_modules/css-loader!./node_modules/less-loader/dist/cjs.js!./src/component/h1/index.less 195 bytes [built]
    + 3 hidden modules
Child extract-text-webpack-plugin /Users/jianchunwang/Workspaces/github-me/learn-webpack/node_modules/extract-text-webpack-plugin/dist /Users/jianchunwang/Workspaces/github-me/learn-webpack/node_modules/css-loader/index.js!/Users/jianchunwang/Workspaces/github-me/learn-webpack/node_modules/less-loader/dist/cjs.js!/Users/jianchunwang/Workspaces/github-me/learn-webpack/src/component/h1/index.less:
       [0] ./node_modules/css-loader!./node_modules/less-loader/dist/cjs.js!./src/component/h1/index.less 195 bytes {0} [built]
        + 1 hidden module
Done in 2.74s.
```

这个时候，我们可以尝试一下 LESS 的特殊性，比如变量、[mixin](http://lesscss.org/features/#mixins-feature) 等来验证 LESS 工作是否符合我们的预期。

比如对 _index.less_ 做如下修改：

```less
#mixin {
  .gradient-l2r(@from; @to) {
    background-image: linear-gradient(to right, @from, @to);
  }
}

.h1 {
  color: fadeOut(#f00, 50);
  flex: 1;
  #mixin.gradient-l2r(#FF0; #FFF);
}
```

为证明 less 是工作的，这里加入了 less 的内置方法 `fadeOut` 和 mixin。

> 注意，这里仅仅是为了说明问题，正式开发的情况下，应该把 mixin 单独成文件。

构建查看效果：

```css
.h1 {
  color: rgba(255, 0, 0, 0.5);
  flex: 1;
  background-image: linear-gradient(to right, #FF0, #FFF);
}
```

# PostCSS

前面我故意留了浏览器兼容性的问题，`linear-gradient`（属性值） 和 `flex`（属性名） 并不是所有浏览器都支持。

> 严格来说，`rgba` 也是，不过 postcss 貌似没有处理它（可能我还不知道）。

你或许可以说「我可以在 mixin 中补全」。嗯，这是一个办法，也是一直以来我用了很久的方式。不过，我们用一种更加工程化、工具化的方式，你会喜欢的。因为，在开发过程中，我们只需要关心标准就行了，兼容性又是什么鬼。

我们需要 [postcss-loader](https://webpack.js.org/loaders/postcss-loader/)：

> PostCSS 并非「在 CSS 之后干什么什么」的意思，你可以把它看成 CSS 界的 babel。有关 PostCSS，你可以看看
> [An Introduction to PostCSS](https://www.sitepoint.com/an-introduction-to-postcss/) 和 [Webdesign
PostCSS Deep Dive](https://webdesign.tutsplus.com/series/postcss-deep-dive--cms-889)

安装 postcss-loader：

```bash
yarn add postcss-loader --dev
```

修改 _webpack.config.js_ 中的 CSS 配置：

```js
{
  test: /\.(css|less)$/,
  use: ExtractTextPlugin.extract({
    use: ["css-loader", "postcss-loader", "less-loader"], // 夹在中间
    fallback: "style-loader" // use style-loader in development
  })
}
```

> Use it after css-loader and style-loader, but before other preprocessor loaders like e.g sass|less|stylus-loader, if you use any.
>
> [wepback postcss-loader 文档](https://webpack.js.org/loaders/postcss-loader/#config-cascade)

这个时候，如果执行 `yarn build` 会报错说「Error: No PostCSS Config found」。在项目根目录下添加文件 _postcss.config.js_，内容如下：

```js
module.exports = {
  plugins: {
    autoprefixer: {
      browsers: ["last 2 versions", "Firefox >= 20"]
    }
  }
};
```

> 关于 browsers 属性怎么写，参考 [browserslist](https://github.com/ai/browserslist)，autoprefixer 就是用的它。

可以 build 了吗？别急。如果你看了 [An Introduction to PostCSS](https://www.sitepoint.com/an-introduction-to-postcss/) 的话，你就应该知道，PostCSS 其实很轻，而它所能完成的炫酷技能其实是靠它的插件得来的。我们这次的任务是「给我增加浏览器属性前缀」，于是我们需要 `autoprefixer` 这个插件（从 _postcss.config.js_ 也应该知道了）：

```bash
yarn add autoprefixer --dev
```

构建结果：

```css
.h1 {
  color: rgba(255, 0, 0, 0.5);
  -webkit-box-flex: 1;
     -moz-box-flex: 1;
      -ms-flex: 1;
          flex: 1;
  background-image: -webkit-gradient(linear, left top, right top, from(#FF0), to(#FFF));
  background-image: linear-gradient(to right, #FF0, #FFF);
}
```

可以看到不论是属性值还是属性名都加上了前缀。等等，怎么 `-moz-linear-gradient` 没有？所以，这就是工程化的工具相较于传统的用 mixin 来做兼容性的好处了，可以省掉可能永远都用不到的代码。

# 总结

这一节，我们了解了：

1. 让 webpack 知道如何处理原生的 CSS 文件（css-loader，style-loader）
2. 把 CSS 抽取成单个的文件并输出
3. 使用 CSS 的超集 LESS 来编写 CSS
4. 使用 PostCSS 来处理一些兼容性问题

打个 tag 先：

```bash
git add .
git commit -m 'CHORE css'
git push
git tag 0.0.4
git push origin 0.0.4
```

代码参考：<https://github.com/justnewbee/learn-webpack/tree/0.0.4>

**打完收工**
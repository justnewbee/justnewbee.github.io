---
layout: post
title: webpack 学习 1 - 从零开始
date: 2017-07-21 22:50:41
categories: develop
tags: js, webpack
---

这一篇，我们将了解：

* webpack 会把多个 JS 文件打包到一个文件
* webpack 原生支持 es6/7 的 `import` 和 `export`
* webpack 的基本构建命令
* webpack 的输出及说明
* webpack 构建结果的基本长相

---

先设定一个小目标：`src` 下有多个 JS 文件，入口文件目前只有一个 `index.js` 想拼成一个文件 `dist/index.js`。

# 创建并初始化项目

创建一个项目，名字叫 `learn-webpack`：

```bash
# 在当前目录下新建目录，作为项目目录
mkdir learn-webpack
# 进去
cd learn-webpack
# 初始化
yarn init -y
# 在该项目中安装 webpack 并把 webpack 写入 package.json 的 devDepencies 下`
yarn add webpack --dev
```

OK，此时目录下的文件结构应该是这样的：

```
\
├── node_modules/
├── package.json
└── yarn.lock
```

**yarn.lock**

是的，你注意到了一个看似「系统生成、并无源码价值」的 _yarn.lock_。而且如果你用 npm5 的话，也会同样生成一个 _package-lock.json_ 的文件，并且郑重其事地跟你说：

> npm notice created a lockfile as package-lock.json. You should commit this file.

你可以在 [npm 的文档](https://github.com/npm/npm/blob/latest/doc/files/package-lock.json.md) 上找到原因。

虽然 yarn 并没有提示你，但它的文档上 [Migrating from npm](https://yarnpkg.com/en/docs/migrating-from-npm) 也说了你需要提交该文件：

> You don't need to read or understand this file - just check it into source control. 

`yarn init` 生成的 _package.json_ 如下（`npm init` 生成的版本会有所差别）：

```json
{
  "name": "learn-webpack",
  "version": "1.0.0",
  "main": "index.js",
  "license": "MIT",
  "devDependencies": {
    "webpack": "^3.1.0"
  }
}
```

好习惯 I，加个 `README.md` 和 `CHANGELOG.md`：

```bash
echo "learn-webpack\n===\n\nlearn webpack step by step" > README.md
echo "CHANGELOG\n===" > CHANGELOG.md
```

好习惯 II，加 [.eslintrc](https://github.com/justnewbee/learn-webpack/blob/master/.eslintrc) 和 [.eslintignore](https://github.com/justnewbee/learn-webpack/blob/master/.eslintignore)，这两个文件。

# 初始化 git

虽然不是必需的，但有个版本控制（哪怕仅仅在本地）也是不错的对吧。在这之前我们需要新建一个 `.gitignore` 文件，以免不必要的文件进入版本控制。

```bash
touch .gitignore
```

并给 `.gitignore` 加上以下内容：

```bash
# common

.*
!.*ignore
!.*rc

# dev

node_modules/

# generated

*.log
dist/
```

好了，接下来就可以初始化 git 而不用担心「误伤无辜」了：

```bash
git init
git add .
git commit -m 'CHORE init'
```

这个时候（或事先也可），去你的 github 上建一个 repo，比如我的就叫 _learn-webpack_，然后把本地的项目提交到 github：

```bash
git remote add origin git@github.com:justnewbee/learn-webpack.git
git push origin master
```

# 添加项目文件

给它加一些目录和文件吧：

```bash
mkdir src
touch src/index.js
touch src/component.js
touch index.html
```

加完之后文件目录结构应该是这样的：

```
\
├── node_modules/
├── src
|   ├── component.js
|   └── index.js
├── .gitignore
├── CHANGELOG.md
├── README.md
├── index.html
├── package.json
└-- yarn.lock
```

接下来是给这些个新建的文件添加内容。

## src/component.js

```js
export default (text = "hello webpack") => {
  const element = document.createElement("div");
  
  element.innerHTML = text;
  
  return element;
};
```

## src/index.js

```js
import component from "./component";

document.body.appendChild(component());
```

你可能已经注意到 ES6 的 `import` 和 `export`（模块的语法）。事实上，webpack 2 开始就已经 [原生支持了这些特性](https://webpack.js.org/api/module-methods/#es6-recommended-)，但仅此而已，其他的 ES6/7 特性还需要借助 [babel](http://babeljs.io)，这个后面【TODO 链接】会讲到。

## index.html

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8" />
<title>learn-webpack</title>
</head>
<body>
<script src="./dist/index.js"></script>
</body>
</html>
```

所以看到它引用了一个不存在的文件 _dist/index.js_，这个正是我们接下来要构建出来的文件。

# 构建，first blood

先尝试用最原始的方式，在当前项目目录下输命令：

```bash
./node_modules/.bin/webpack src/index.js dist/index.js
```

这个命令说：哎，把 _src/index.js_ 这个入口文件和它关联的任何模块都打包到 _dist/index.js_。

你会看到如下输出：

```bash
➜  learn-webpack git:(master) ./node_modules/.bin/webpack src/index.js dist/index.js
Hash: d260ac6928845823c460
Version: webpack 3.3.0
Time: 101ms
   Asset     Size  Chunks             Chunk Names
index.js  3.08 kB       0  [emitted]  main
   [0] ./src/index.js 77 bytes {0} [built]
   [1] ./src/component.js 148 bytes {0} [built]
```

构建成功，好的开端!

对输出内容稍作说明：

* `Hash: d260ac6928845823c460` - 此次构建的哈希值，可用于替换在构建脚本中的 `[hash]` 替换符
* `Version: webpack 3.3.0` - 不解释
* `Time: 101ms` - 构建过程总耗时
* `index.js  3.08 kB       0  [emitted]  main`
    - `index.js` - 构建的目标文件名
    - `3.08 kB` - 文件体积
    - `0` - 与之关联的代码片段的 ID
    - `[emitted]` 创建方式，为输出
    - `main` 代码片段名称
* `[0] ./src/index.js 77 bytes {0} [built]`
    - `[0]` - 当前代码片段的 ID（在构建出的文件中可以找到对应的区块注释类似 `/* 0 */`）
    - `./src/index.js` - 文件路径
    - `77 bytes` - 文件体积
    - `{0}` - 入口代码片段的 ID
    - `[built]` - 创建方式，为仅构建

到此，webpack 的配置已经好了，用起来也蛮方便，接下来我们来探索一下更多的功能吧。

此时可以去看看构建出来的 _dist/index.js_ 长啥样（为了篇幅，省略了一些）：

```js
/******/ (function(modules) {
/******/   // ... webpackBootstrap
/******/ })
/******/ ([
/* 0 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {
"use strict";
Object.defineProperty(__webpack_exports__, "__esModule", { value: true });
/* harmony import */ var __WEBPACK_IMPORTED_MODULE_0__component__ = __webpack_require__(1);
document.body.appendChild(Object(__WEBPACK_IMPORTED_MODULE_0__component__["a" /* default */])());
/***/ }),
/* 1 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {
"use strict";
/* harmony default export */ __webpack_exports__["a"] = ((text = "hello webpack") => {
  const element = document.createElement("div");
  element.innerHTML = text;
  return element;
});
/***/ })
/******/ ]);
```

对于 webpack 命令行的输出，可以看出以下几点：

1. 构建得到的 JS 代码中注释着 `/* 0 */` 和 `/* 1 */` 跟构建日志中的 `[0] ./src/index.js` 和 `[1] ./src/component.js` 是对应的
2. ES7 的 `import` 和 `export` 语法已经被 webpack 内置支持
3. 构建出来的代码自动被加上了 `"use strict"`，这是因为 ES6/7 的模块定义默认是严格模式的
4. 其他的 ES6/7 特性似乎并没有被支持，例如参数默认值、箭头函数、`const` 等都没有被编译，也就是说，目前构建出来的代码只有「好」浏览器才能玩转

# 运行 index.html

如果 _index.html_ 在浏览器中展示 `hello webpack` 的文案，那么就说明 webpack 的构建彻底成功了。

打开它的方式有很多种，你可以任选以下几种：

* Finder 中直接用浏览器打开
* 直接在 terminal 中用命令打开 `open index.html`（当前目录下）
* 如果你用的是 WebStorm 或 IntellijIDEA，你可以右键然后 _Open in Browser_
* 利用 python 在项目目录下起服务器 `python -m SimpleHTTPServer 8080`，然后访问 `http://127.1:8080`
* 如果你全局安装过 [http-server](https://www.npmjs.com/search?q=http-server)（`npm install -g http-server`），你也可以在项目根目录下起服务器 `http-server`，然后访问 `http://127.1:8080`（它的默认端口是 `8080`）
* 利用 apache、jetty、tomcat、nginx...这些会稍稍麻烦一些
* …

用 Firefox、Chrome 或其他「好浏览器」打开 _index.html_，结果当然是没问题的，你可以看到有 **hello webpack** 这样的文案、

# 总结

到此，就 OK… 了？NO No no，其实这里还蛮多问题的：

* 命令太长，且带有文件参数，记不住，也不灵活
* JS 中的 es6/7 特性不能被所有的浏览器接受
* 如果要加入 CSS 呢？
* 如果 component 越来越多呢？
* 如果有多个入口文件要输出呢？
* _index.html_ 中的 _dist/index.js_... 严格来说它是不存在的，因为 _dist_ 被忽略了，不会提交
* …

但本篇的成就已经达成 - 完成多个 JS 文件打包成一个文件，并且打包出来的代码是（在「好」浏览器内）可用的。后续的文章来慢慢破解这些问题，同时对 webpack 的更多特性进行研究和实践。

在这之前，代码可以告一段落了，打个 tag 先：

```bash
git add .
git commit -m 'CHORE steup webpack for basic usage'
git push
git tag 0.0.1
git push origin 0.0.1
```

代码参考：<https://github.com/justnewbee/learn-webpack/tree/0.0.1>

**打完收工**
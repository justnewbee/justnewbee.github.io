---
layout: post
title: webpack 学习 1 - 从零开始
date: 2017-07-22 10:50:00
categories: develop
tags: js, webpack
---

这一篇，我们会了解：

* 利用 _npm scripts_ 和 `webpack.config.js` 简化每次命令的敲打
* 一个最简单的 `webpack.config.js` 用例

---

目标，添加配置文件，简化命令调用。毕竟每次输入 `./node_modules/.bin/webpack src/index.js dist/index.js` 这么一长串确实挺扯淡的。

在这之前我们先修改一下 _package.json_ 中的版本号：

```bash
yarn version --new-version 0.0.2
```

# 去参

我们先把命令简化一下，不妨去掉里面的参数，因为这些文件路径参数导致的问题就是「不灵活」，这在我看来是个很要命的问题。如果我们直接去掉这些参数，webpack 就迷乱了：

```bash
➜  ./node_modules/.bin/webpack
No configuration file found and no output filename configured via CLI option.
A configuration file could be named 'webpack.config.js' in the current directory.
Use --help to display the CLI options.
```

webpack 说「请给我一个叫 `webpack.config.js` 的文件」。是的，webpack 默认会找当前项目下有没有这个文件，有的话就会用里面的配置。那么给一个咯：

```bash
touch webpack.config.js
```

以下是该文件的内容：

```js
const path = require("path");

module.exports = {
  entry: "./src/index.js",
  output: {
    path: path.resolve(__dirname, "dist"),
    filename: "index.js"
  }
};
```

这里定义了 [entry](https://webpack.js.org/concepts/entry-points/) 和 [output](https://webpack.js.org/concepts/output/)。前者定义了 webpack 需要进行扫描的起点，后者定义了打包输出的文件。

这个时候再执行无参的命令就没问题了：

```bash
➜  ./node_modules/.bin/webpack
Hash: d260ac6928845823c460
Version: webpack 3.1.0
Time: 105ms
   Asset     Size  Chunks             Chunk Names
index.js  3.07 kB       0  [emitted]  main
   [0] ./src/index.js 77 bytes {0} [built]
   [1] ./src/component.js 148 bytes {0} [built]
```

> webpack 可以通过参数 `--config` 指定不同的配置文件，只不过默认文件是 `webpack.config.js`，即命令 `webpack` 等价于 `webpack --config webpack.config.js`。
> 
> 详见 <https://webpack.js.org/configuration/>

# 去掉命令路径前缀

为什么敲命令必须带上 `./node_modules/.bin` 的路径呢？还是麻烦。那么直接打 `webpack` 可不可以呢？答案是「可以，但也不可以」。

如果你全局安装了 webpack，那么这么做可能 OK，但如果你并未全局安装，则就会说命令找不到。但如果我们用 [npm scripts](https://docs.npmjs.com/misc/scripts) 来执行 `webpack` 命令就另当别论了，因为它会系统 `PATH` 下给你加上当前项目的 `./node_modules/.bin`，所以在 _npm script_ 下的命令可以直接写 `webpack`。

> 具体见 [npm run script](https://docs.npmjs.com/cli/run-script) 的文档。

给 `package.json` 添加如下代码片段：

```json
"scripts": {
  "build": "webpack"
}
```

然后执行 `yarn build`：

```bash
➜  yarn build
$ webpack
Hash: 7b857d818b5f3fb48aca
Version: webpack 3.1.0
Time: 150ms
   Asset     Size  Chunks             Chunk Names
index.js  3.07 kB       0  [emitted]  main
   [0] ./src/index.js 77 bytes {0} [built]
   [1] ./src/component.js 148 bytes {0} [built]
```

`yarn build` 等同于 `yarn run build`，但如果用 _npm_，则必须为 `npm run build`，命令行输出稍有不同。

# 总结

解决了第一篇中的第一个问题：命令太长，且带有文件参数，记不住，也不灵活。

同样，打个 tag 先：

```bash
git add .
git commit -m 'CHORE use npm scripts and webpack.config.js to ease the keyboard pain'
git push
git tag 0.0.2
git push origin 0.0.2
```

代码参考：<代码参考：<https://github.com/justnewbee/learn-webpack/tree/0.0.12>

**打完收工**
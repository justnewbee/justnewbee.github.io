---
layout: post
title: webpack 学习 6 - 开发自动化工具
date: 2017-10-31 23:48:27
categories: develop
tags: js, webpack
---

这一篇的目标是——

了解如何借助开发工具实现更自动的开发工作，脏活累活交给工具，而我们只需要关心业务的核心代码即可。

希望解决的问题：

1. 改代码不在需要手动编译，不再需要手动刷新浏览器进行验证
2. HTML 里面不应该存在没放到 repo 中的文件路径，最好整个 dist 目录都可以不存在，眼不见为静

---

通过这一篇，我们将了解：

* 监视代码修改，自动编译
* 让 HTML 可以加载编译后的文件，且编译后的文件不需要输出到实际的目录（也就没有必要提交到仓库）
* 利用工具起服务，而不是手工敲外部命令，并且在代码更新后自动刷新

---

修改版本号：`yarn version --new-version 0.0.6`。

# 手动之痛

到现在，我们所做的「开发」工作，都还是纯手工：

1. 写一个 HTML，手工加入构建后的路径（并不存在于代码仓库）
2. 用 IDE 或其他方式手工起服务，或直接用浏览器打开 HTML 文件
3. 修改代码
4. 执行命令 `yarn build` 进行构建
5. 手动刷新浏览器
6. 周而复始...

偶尔这么搞搞是可以忍受的，但实际开发过程中，这是很糟糕的体验，效率低下且烦人。

webpack 的文档 [Choosing a Development Tool](https://webpack.js.org/guides/development/#choosing-a-development-tool) 对此做了简单的说明：

1. `watch`
2. [webpack-dev-server]
3. [webpack-dev-middleware]

# watch

很多工具都会有 **watch** 功能，参数一般是 `--watch` 或简写的 `-w`，webpack 也不例外，git it a shot：

```bash
➜  node_modules/.bin/webpack -w

Webpack is watching the files…

...
```

所以你会看到「Webpack is watching the files…」这句话，并且，在构建结果输出后，进程并不会结束。这个时候，如果你去修改 `src` 下面的代码，就会发现控制台会有相应的输出。

> 你修改的代码必须要是 webpack 在构建过程中使用到的。如果你新加了一个模块，而这个模块不被任何人所引用，即使你再怎么改这个文件，webpack 对此完全会置之不理。

在 _package.json_ 中 `scripts` 下添加命令 `"watch": "webpack -w"`，然后执行 `yarn watch` 或 `npm run watch` 的效果是一样的，不赘述。

`watch` 后，只要代码有改动，控制台输出就会增加，出错不容易查。它只解决了代码改动后自动编译的问题，并不能解决其他问题：

* 浏览器并不知道代码有改动，我们还需要手动刷新浏览器
* _index.html_ 还是需要手工起服务
* _index.html_ 中加载的资源虽然本地有，但代码仓库中却已经被忽略，也就是一个新 clone 下来的代码，_index.html_ 是有错误的，因为它引用了不存在的文件

# dev server

`watch` 让我们在修改代码的时候不再需要重复执行 `yarn build`，但我们还是需要手动刷新测试页面。

跟着我们来试试 [webpack-dev-server]，它可以帮我们做「live reloading」，安装：

```bash
yarn add webpack-dev-server --dev
```

give it a shot：

```bash
➜  node_modules/.bin/webpack-dev-server
Project is running at http://localhost:8080/
webpack output is served from /
ts-loader: Using typescript@2.4.2 and /Users/jianchunwang/Workspaces/github-me/learn-webpack/tsconfig.json
Hash: 218f7332b3fe27691651
Version: webpack 3.5.5
Time: 3788ms
    Asset       Size  Chunks                    Chunk Names
 index.js     325 kB       0  [emitted]  [big]  main
index.css  357 bytes       0  [emitted]         main
  [35] multi (webpack)-dev-server/client?http://localhost:8080 ./src/index.js 40 bytes {0} [built]
  [36] (webpack)-dev-server/client?http://localhost:8080 5.83 kB {0} [built]
  [37] ./node_modules/url/url.js 23.3 kB {0} [built]
  [43] ./node_modules/strip-ansi/index.js 161 bytes {0} [built]
  [44] ./node_modules/ansi-regex/index.js 135 bytes {0} [built]
  [45] ./node_modules/loglevel/lib/loglevel.js 6.74 kB {0} [built]
  [46] (webpack)-dev-server/client/socket.js 856 bytes {0} [built]
  [78] (webpack)-dev-server/client/overlay.js 3.6 kB {0} [built]
  [79] ./node_modules/ansi-html/index.js 4.26 kB {0} [built]
  [80] ./node_modules/html-entities/index.js 231 bytes {0} [built]
  [83] (webpack)/hot nonrecursive ^\.\/log$ 170 bytes {0} [built]
  [85] (webpack)/hot/emitter.js 77 bytes {0} [built]
  [86] ./node_modules/events/events.js 8.33 kB {0} [built]
  [87] ./src/index.js 332 bytes {0} [built]
  [88] ./src/component/h1/index.ts 289 bytes {0} [built]
    + 79 hidden modules
Child extract-text-webpack-plugin node_modules/extract-text-webpack-plugin/dist node_modules/css-loader/index.js!node_modules/postcss-loader/lib/index.js!node_modules/less-loader/dist/cjs.js!src/component/h1/index.less:
       [0] ./node_modules/css-loader!./node_modules/postcss-loader/lib!./node_modules/less-loader/dist/cjs.js!./src/component/h1/index.less 539 bytes {0} [built]
       [1] ./node_modules/css-loader/lib/css-base.js 2.26 kB {0} [built]
webpack: Compiled successfully.
```

[webpack-dev-server] 是一个小型的 [express] 服务器，它使用 [webpack-dev-middleware] 来服务于 webpack 的包，除此自外，并利用 socket 进行 live reloading。

[webpack-dev-server] 在以当前项目根目录为 root，在 8080 端口上起了一个服务，这时访问 `http://localhost:8080/`，你**可能**会发现跟原来直接访问 _index.html_ 的效果是一样的。

所以，你可能迫不及待地去改 _src_ 下面的文件，想看看是不是可以「live reloading」，然而，你肯定会失望。之所以说「可能」，是因为我们前面已经把代码构建到了 _dist_ 目录。

你会注意到，当修改 _src_ 下面的文件的时候，控制台会告诉你它进行了编译，然而，dist 下面却没有动静。

_src/index.html_ 引了 dist 下面的文件，所以是不生效的。这时候你需要 [devServer.publicPath](https://webpack.js.org/configuration/dev-server/#devserver-publicpath-)。

> The bundled files will be available in the browser under this path.

## 配置

在 _package.json_ 的 `scripts` 下添加 `"start": "webpack-dev-server --open"`。

在 _webpack.config.js_ 中增加 `devServer`：

```js
devServer: {
  publicPath: "/dist/"
}
```

如果，_dist_ 目录还在，`rm -rf dist` 删掉它。

接着，`yarn start`，会自动打开默认浏览器到 `http://localhost:8080/`，能正常工作；更棒的是（有时候会比较头疼），当 _src_ 下的文件改动的时候，除了会自动编译之外，浏览器还会自动 reload。简单看一下浏览器的 Network，可以看到它其实建立了一个 websockit 链接，从 console 输出的编译出的 JS 文件中，也可以看到 websocket。

看起来不错！在没有 _dist_ 目录的情况下，_index.html_ 工作得很棒。

不过问题在于，我还是要在 _index.html_ 中写两个不存在的路径：`./dist/index.js` 和 `./dist/index.css`。

于是就到了 [html-webpack-plugin] 出场的时候了。

```bash
yarn add html-webpack-plugin --dev
```

然后在 _webpack.config.js_ 稍作修改：

```js
// 前面加上
const HtmlWebpackPlugin = require("html-webpack-plugin");

// 后面的 plugins 数组加上，注意这里的配置参数只是为了说明一些问题，并非必需
new HtmlWebpackPlugin({
  title: "My App",
  filename: "my/app.html"
})
```

然后执行 `yarn start`，访问 `http://localhost:8080/dist/my/app.html` 会发现跟访问 `http://localhost:8080/` 如出一辙，区别在于，两个 title 不一样。

查看一下源码，是这样的：

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>My App</title>
  <link href="../index.css" rel="stylesheet"></head>
  <body>
  <script type="text/javascript" src="../index.js"></script></body>
</html>
```

这里面的内容都是由 [html-webpack-plugin] 自动生成的。

回头看一下我们的项目目录，并没有实质的 _dist/my/app.html_ 生成，这个得益于我们用了 [webpack-dev-server]，如果我们用 `yarn build` 进行构建，这些文件都会被构建出来。

所以，[html-webpack-plugin] 的作用就是在 webpack 的 `output` 配置指定的 `path` 目录下创建一个 `filename` 指定的文件（默认为 _index.html_），把构建出的 JS 和 CSS 安插进去，并保证文件路径的正确。

通过 [配置参数](https://github.com/jantimon/html-webpack-plugin#configuration)，可以配置 HTML 的模板，这样的话，项目根目录下的 _index.html_ 基本上就可以功成身退了，改造一下。

```bash
mkdir demo
git mv index.html demo/.
```

把 `index.html` 移入 `demo` 目录有这样的好处：

1. 一看就知道这是用于本地 demo 的，不需要特殊关心
2. `demo` 目录可以支持多个文件，满足不同的需求，项目根目录永葆清爽
3. `demo` 目录可以直接在 `.npmignore` 文件中进行统一忽略

因为 [html-webpack-plugin] 会给 HTML 自行添加构建完成的，这里需要对其内容稍作修改，去掉原先硬编码的（不存在于仓库记录）的前端资源：

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8" />
<title>learn-webpack</title>
</head>
<body>
</body>
</html>
```

修改 _webpack.config.js_ 中的 `plugin` 部分：

```bash
new HtmlWebpackPlugin({
  template: "demo/index.html"
})
```

以及 `devServer.publicPath` 由原来的 `/dist` 改成 `/`：

```
devServer: {
  publicPath: "/"
}
```

其实，这里 `HtmlWebpackPlugin` 的 `template` 属性都是可以没有的，也就是说，我们可以完全去掉 `demo/index.html` 这个文件，这套代码照样可以很好的工作。

很棒是不是，有了这些，开发期间用到的 demo 代码可以跟实际代码做到很好的隔离。

# webpack-dev-middleware

[webpack-dev-middleware] 是 [webpack-dev-server] 的依赖，但你也可以用它来做一些 low-level 的自定义。

简言之，它是给「高级」用户用的，一般来说，[webpack-dev-server] 提供的立即可用的解决方案已经够大部分人使用了。

# 总结

这一节，我们：

1. 利用了 webpack 的 watch 进行代码的自动编译
2. 利用 [webpack-dev-server] 把自动编译，live-reloading 等开发必需的功能串了起来
3. 让 index.html 得到更佳的居所，并不需要在里面引用不存在于仓库的文件，保证整个仓库的代码是「有据可查」的，从而是干净的

打个 tag 先：

```bash
git add .
git commit -m 'CHORE dev-server'
git push
git tag 0.0.6
git push origin 0.0.6
```

代码参考：<https://github.com/justnewbee/learn-webpack/tree/0.0.6>

**打完收工**

[webpack-dev-server]: https://github.com/webpack/webpack-dev-server
[webpack-dev-middleware]: http://webpack.github.io/docs/webpack-dev-middleware.html
[html-webpack-plugin]: https://github.com/jantimon/html-webpack-plugin
[express]: http://www.expressjs.com.cn/
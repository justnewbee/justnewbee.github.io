---
layout: post
title: webpack 学习 0
date: 2017-07-14 22:19:41
categories: develop
tags: js
---

用了 webpack 也好久了，觉得这个东西真的是好，解决了之前觉得很难受但又不得不容忍的很多问题，比如之前样式代码和 JS 就是分开的，必须靠全局搜索才能找到某个样式在哪里被使用。

呃，好吧，不过其实我真的不会用 webpack... 所以得好好的学习一下，同时把过程记录一下，整理出一个系列的文章。

在此系列文章中，涉及到的环境和工具如下：

* mac（如果 Windows，可能会有所区别）
* node ≥ 6.9.0
* yarn ≥ 0.27.5 或 npm ≥ 5.1.0
* webpack 3.0

> 官方说 webpack 2 升级到 3 几乎可以不用改，但从 1 升级到 2 的改动就要一些了。参考 [Migrating from v1 to v2](https://webpack.js.org/guides/migrating/)



现下貌似 [yarn](https://yarnpkg.com/) 越来越流行了，没怎么用过，不过我会在本文中优先使用 yarn。

如果你还是倾向于用 [npm](https://www.npmjs.com/)，你可以继续用 npm，一般来说 yarn 的命令很像 npm，可能更简洁一些，以下是后面会用到的两个版本的命令对照。

yarn | npm | 说明
:-- | :-- | :--
`yarn init` | `npm init` | 初始化 `package.json`，带 `-y` 参数表示「别问我问题，都默认」，yarn 命令生成的文件简洁一些，license 也不一样
`yarn start` | `npm start` 或 `npm run start` | 执行 npm scripts 之 `start`
`yarn build` 或 `yarn run build` | `npm run build`（然不可 `npm build`） | 执行 npm scripts 之 `build`
`yarn add xxx` | `npm install xxx --save` 或 `npm i xxx --save` | 安装 package
`yarn add xxx --dev` 或 `yarn add xxx -D` | `npm install xxx --save-dev`或 `npm install xxx -D` | 安装 package，并存到 `devDependencies`

> 后面用到的 yarn 命令，你都可以用相应的 npm 命令替换。

# webpack 是什么

# webpack 不是什么

# webpack 适用场景

# webpack 命令

# webpack config

# webpack 的基本概念

## entry point

## output

## loader

<https://webpack.js.org/loaders>

loader 做什么

怎么用

以下是官方文档中列出的一系列 loader。

### babel-loader

用于 xxx

配置方式

### bundle-loader

用于 xxx

配置方式

### cache-loader

用于 xxx

配置方式

### coffee-loader

用于 xxx

配置方式

### coffee-redux-loader

用于 xxx

配置方式

### coverjs-loader

用于 xxx

配置方式

### css-loader

用于 xxx

配置方式

### exports-loader

用于 xxx

配置方式

### expose-loader

用于 xxx

配置方式

### extract-loader

用于 xxx

配置方式

### file-loader

用于 xxx

配置方式

### gzip-loader

用于 xxx

配置方式

### html-loader

用于 xxx

配置方式

### i18n-loader

用于 xxx

配置方式

### imports-loader

用于 xxx

配置方式

### istanbul-instrumenter-loader

用于 xxx

配置方式

### jshint-loader

用于 xxx

配置方式

### json-loader

用于 xxx

配置方式

### json5-loader

用于 xxx

配置方式

### less-loader

用于 xxx

配置方式

### mocha-loader

用于 xxx

配置方式

### multi-loader

用于 xxx

配置方式

### node-loader

用于 xxx

配置方式

### null-loader

用于 xxx

配置方式

### postcss-loader

用于 xxx

配置方式

### raw-loader

用于 xxx

配置方式

### react-proxy-loader

用于 xxx

配置方式

### restyle-loader

用于 xxx

配置方式

### sass-loader

用于 xxx

配置方式

### script-loader

用于 xxx

配置方式

### source-map-loader

用于 xxx

配置方式

### style-loader

用于 xxx

配置方式

### svg-inline-loader

用于 xxx

配置方式

### thread-loader

用于 xxx

配置方式

### transform-loader

用于 xxx

配置方式

### url-loader

用于 xxx

配置方式

### val-loader

用于 xxx

配置方式

### worker-loader

用于 xxx

配置方式

### yaml-frontmatter-loader

用于 xxx

配置方式

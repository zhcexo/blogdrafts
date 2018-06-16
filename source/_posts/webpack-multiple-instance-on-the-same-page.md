---
title: 同一页面运行wepback不同实例出现冲突的解决办法
date: 2018-04-18 11:03:00
tags: webpack
---
### 遇到的问题

对于同一个页面功能由不同的同事开发，都用到了 `webpack` 以及 `CommonsChunkPlugin`，最后把打包出来的代码，整合到一起的时候，冲突了。

### 问题表现

各自用 `webpack` 打包代码没有问题，但是加载到页面上时，代码报错且错误难以定位。

### 解决方法

在 `webpack` 的配置选项里使用 `output.jsonpFunction`。

看一下文档里说的：

> output.jsonpFunction
`string`
仅用在输出目标为 web，且使用 jsonp 的方式按需加载代码块时。
一个命名的 JSONP 函数用于异步加载代码块或者把多个初始化代码块合并到一起时使用（如 CommonsChunkPlugin, AggressiveSplittingPlugin）。
当同一个页面上有多个 webpack 实例（源于不同的编译），需要修改这个函数名。
如果使用了 `output.library` 选项，那么这个 `library` 的命名会自动附加上。

事实上 webpack 并不在全局命名空间下运行，但是 `CommonsChunkPlugin` 这样的插件会使用异步 JSONP 的方法按需加载代码块。插件会注册一个全局的函数叫 `window.webpackJsonp`，所以同一个页面上运行多个源自不同 webpack 打包出来的代码时，可能会引起冲突。

### 参考资料：

[webpack - configuration - output - jsonpfunction](https://webpack.js.org/configuration/output/#output-jsonpfunction)

[How to run multiple webpack instances on the same page…and avoid any conflicts](https://medium.com/@cliffers/how-to-run-multiple-webpack-instances-on-the-same-page-and-avoid-any-conflicts-4e2fe0f016d1)
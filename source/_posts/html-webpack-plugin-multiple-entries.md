---
title: 让HTML Webpack Plugin根据多entry把html文件生成到指定目录中
date: 2018-06-16 22:13:40
tags: webpack
---
### 遇到的问题

项目里有需要处理一堆目录的 html 文件自动生成，并且生成到相应的目录中。与 html 文件相对应的样式 css 文件和脚本 js 文件，也要放到相应的目录中。例如：

```
目录结构如下：
- dist
- src
    |- a
        `- index.js
    |- b
        `- index.js
    |- c
        `- index.js
    `- tpl
        `- index.html
    
希望得到的结果是，a，b，c 三个目录中的 index.js 都是入口文件，以 tpl 中的 index.html 为模板，
经由 webpack 打包之后，在 dist 目录中生成 a，b，c 三个目录，
并且每个目录中都存在 html，css，js 三个目录，里面存放相应生成的文件。
生成如下的目录结构：
- dist
    |- a
        |- css
        |- html
        `- js
    |- b
        |- css
        |- html
        `- js
    `- c
        |- css
        |- html
        `- js
- src
```

### 解决方案

说到自动生成 html 文件，第一想法就是 `HtmlWebpackPlugin`，但是这个插件不支持 webpack 中类似于 `[name]` 这样的命名，所以想用 `[name]` 这样的命名方式变向成生文件夹的方法是行不通的。另外，每个条 `new HtmlWebpackPlugin()` 语句，只能生成一个页面，像我这样要大批量生成文件，是不太方便的。

因为要做到生成的文件，有对应的目录，所以在配置 webpack 的 entry 的时候，可以使用多入口方式。

大体代码如下：

```javascript
var entries: {
    a: './a',
    b: './b',
    c: './c'
};

var entryHtmlPlugins = Object.keys(entries).map(function(entryName) {
    return new HtmlWebpackPlugin({
        filename: entryName + '.html',
        chunks: [entryName]
    });
});

module.export = {
    entry: entries,
    //....
	plugins: [
	// ..
	].concat(entryHtmlPlugins)
}
```

### 参考资料
[HTML Webpack Plugin](https://github.com/jantimon/html-webpack-plugin)
[Support multiple Webpack entries](https://github.com/jantimon/html-webpack-plugin/issues/299)
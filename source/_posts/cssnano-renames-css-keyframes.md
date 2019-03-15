---
title: cssnano压缩了CSS动画中keyframes的名字
date: 2019-03-15 20:04:18
tags: css
---
最近在迁移一些老代码，移动端有一个 loading 动画是用 CSS3 写的，正常得不得了。可是把它集成到 webpack 使用的 scss 里之后，动画效果就丢掉了。找到 scss 转换出来的 css，查找出问题的代码，发现里面的 `@keyframes bouncedelay` 变成了 `@keyframes a`，查看了一下 webpack 的配置文件，把问题原因锁定到 cssnano 上。

首先是找到了 cssnano 的官网，配置项很多，感觉一个个看下来太浪费时间，直接就找问题本身。于是乎在 github 上找到了一个 issue，按别人提供的方法，果然解决了问题。

修改 webpack 配置，方法如下：

```javascript
// 修改之前
{
    loader: 'postcss-loader',
    options: {
        plugins: (loader) => [
            require('autoprefixer')(),
            require('cssnano')({
                zindex: false
            })
        ]
    }
}
```

```javascript
// 修改之后
{
    loader: 'postcss-loader',
    options: {
        plugins: (loader) => [
            require('autoprefixer')(),
            require('cssnano')({
                zindex: false,
                reduceIdents: false
            })
        ]
    }
}
```

修改 webpack 的配置之后再试，打包出来的代码中 bouncedelay 就没有变成 a 了。

补充说明：

- cssnano 的 `reduceIdents` 选项，是用于去掉 css 中 “用户自定义字符串标识符” 的 postcss 插件
- 由于不同方式处理出不同的 css 样式表，可能会出现在同一个网页中，所以也可能发生 keyframes 这个名相互覆盖的问题，所以建议在使用 cssnano 的时候，始终将这个选项置为 `false`。

## 参考资料

- [cssnano renames css keyframes to "a"](https://github.com/cssnano/cssnano/issues/247)
- [postcss-reduce-idents](https://github.com/ben-eb/postcss-reduce-idents)
- [MDN 上 <custom-ident> 的介绍](https://developer.mozilla.org/zh-CN/docs/Web/CSS/custom-ident)

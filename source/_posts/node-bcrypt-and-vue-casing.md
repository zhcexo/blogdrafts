---
title: 最近尝试Vue遇到的坑：安装bcrypt和命令行大小写错误提示
date: 2019-03-08 15:02:32
tags: [node,vue,powershell]
---
开门见山——

### 关于 Vue 的

#### vue-cli

我看的教材上，用的 vue-cli，以前的安装方式就是 `npm install -g vue-cli`，但是新版本变成了 `npm install -g @vue/cli`。

其实[官方的文档](https://cli.vuejs.org/guide/)有说，但是我没有注意而已……老版本的 `vue-cli` 是 v2 版本，新版的 `@vue/cli` 是 v3 版本。

#### npm run dev 时遇到的警告提示

另一个问题是，使用 `vue init` 建立新项目之后，在命令行中运行 `npm run dev` 居然出了黄色的警告提示：**There are multiple modules with names that only differ in casing.** 比较迷的是，我头一天用的时候，是没有任何提示的，第二天回公司，重新运行，就出一大堆提示了。找了一堆资料，大概有两种原因：

> - 用 `import Vue from 'Vue'` 这样的写法引用了模块，应该改成小写的：`import Vue from 'vue'`
> - 使用了会区分大小写的命令行，例如 git bash 或者 powershell

我自己遇到的问题是第二种，头一天用的 cmd 命令行，第二天用的 powershell，所以就出问题了。有遇到相同问题的朋友，可以找找看，是不是这两个原因。

### 关于 node 的

#### 安装 node-gyp

下载了网上的一个测试 node 服务器，用里面的 `package.json` 文件，用 `npm install` 安装之后，不停报错。分析命令行提示，缺少 `node-gyp`。继续在网上找资料，`node-gyp` 是没有整合到 `nodejs` 的安装包里，需要自己编译安装。而要在 windows 平台上编译安装，又需要安装 [`windows-build-tools`](https://github.com/felixrieseberg/windows-build-tools)。

按项目说明来：

```shell
npm install -g --production windows-build-tools
```

然后，然后就特么一直卡在那里不动了……

又找了一堆资料，好像是最新版本的这个工具有问题，安装 `4.0.0` 的就不会出错，那么再来：

```shell
npm install -g --production windows-build-tools@4.0.0
```

等待命令行窗口里面该搞的搞完了，就可以继续输入命令安装 `node-gyp` 了：

```shell
npm install -g node-gyp
```

#### 安装 bcrypt

`bcrypt` 的安装需要依赖 `node-gyp`，所以上面才折腾半天的。但问题是，如果直接按下载的 `package.json` 提供的版本安装 `1.x.x` 版本时，会提示 github 上找不到……

OK，还是安最新的吧。。。

```shell
npm install bcrypt
```

#### 不要忽略 package-lock.json 文件的影响

前不久重装了一下系统，然后安装了一下需要用的工具。在我的工具文件夹里，使用 `npm install` 的时候，总是提示 `base64-url 安装错误，/path/to/base64-url.tar.gz 文件不存在`，而且这个 `path` 是一个本地路径，也是一个无法在 windows 系统里存在的路径。

折腾半天无果，发现是被同目录下的 `package-lock.json` 影响了。因为之前从 `cnpm` 切换到 `npm` 时，生成的 `package-lock.json` 基于了当时安装好的 node 模块，所以路径指向了本地。

我删掉 `package-lock.json` 之后再装新版本，就没这些问题了。当然，有些版本变化较大的工具，还是要卸载后安装对应的旧版，也挺糟心的。

#### 一个好用的工具 nrm

什么是 nrm？

> nrm can help you easy and fast switch between different npm registries, now include: npm, cnpm, taobao, nj(nodejitsu).

就是用来快速切换 npm 源的一个管理工具，里有 npm 原始源地址，也有淘宝和其他的。

```shell
npm install -g nrm
```

安装之后，直接在命令行里用 `nrm ls` 就可以显示已经有的源，和正在使用的源；用 `nrm use [name]` 就能切换到指定源了，例如 `nrm use taobao`。用 `nrm` 来完成这些小动作真的会省心不少，推荐。
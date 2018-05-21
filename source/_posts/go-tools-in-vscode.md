---
title: 在 VSCODE 中安装 Go 语言相关支持
date: 2018-05-21 09:58:03
tags: go
---
写好 Go 文件，用 VSCode 打开之后，右下角会提示缺少 Go 相关的支持，然后会有两个按钮 `Install` 和 `Show`。`Install` 是自动安装这些支持，`Show` 是显示缺了些什么。但实际情况是，点了 `Install` 之后，控制台一堆报错并且安装失败。

点击 `Install` 或者 `Show`，控制台里已经显示缺少如下工具：

```
github.com/ramya-rao-a/go-outline
github.com/acroca/go-symbols
golang.org/x/tools/cmd/guru
golang.org/x/tools/cmd/gorename
github.com/rogpeppe/godef
github.com/sqs/goreturns
github.com/golang/lint/golint
github.com/derekparker/delve/cmd/dlv
```

然后用 `go install` 或者 `go get -v` 都安装不成功。

### 解决办法

1. 打开 Go 的安装目录
2. 进入安装目录下的 `src` 目录，新建 `golang.org` 目录，在此目录下继续新建 `x` 目录
3. 进入 `安装目录/golang.org/x` 目录，执行以下两个命令

```
git clone https://github.com/golang/tools.git
git clone https://github.com/golang/lint.git
```

第一个安装除 `go-lint` 之外其他工具的支持，第二个安装 `go-lint` 支持。

以上步骤都处理完之后，就可以用 `go get ***` 和 `go install ***` 这种方式把缺的工具都装好。安装之后重启 VScode 即使用 Go 语言相关支持。
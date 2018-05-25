---
title: Golang 中为切片 append 元素之后 cap 变化的问题
date: 2018-05-25 14:04:04
tags: go
---
在学习 Go 语言基础的时候，看到了 slice 切片这里，里面有个 append 方法，用于往 slice 里添加元素。代码如下：

```go
package main

import (
    "fmt"
)

func main() {
    var numbers []int
    printSlice(numbers)

    numbers = append(numbers, 0)
    printSlice(numbers)

    numbers = append(numbers, 1)
    printSlice(numbers)

    numbers = append(numbers, 2, 3, 4)
    printSlice(numbers)
}

func printSlice(x []int) {
    fmt.Printf("len=%d cap=%d slice=%v\n", len(x), cap(x), x)
}
```

代码运行的结果是：

```shell
len=0 cap=0 slice=[]
len=1 cap=1 slice=[0]
len=2 cap=2 slice=[0 1]
len=5 cap=6 slice=[0 1 2 3 4]
```

差一点就看看跳过了，不过最后一行的结果，`cap` 执行的结果显示，这个 `numbers` 切片的容量为 6。

如果是写其他的语言，比如 JavaScript，那么这个结果可能会跟 `length` 作类比从而产生“为什么容量不是 5”这种想法，毕竟我们只在 slice 里添加了 5 个元素。翻了一下官方的文档也没有解释，后来在 stackoverflow 上看到了别人的回答，大致总结如下：

> Go 会为你的 slice 提供比你需要的更多的容量，原因是在 slice 的底层，有个不可变动的（immutable）数组（array）在实际起作用。当你要为 slice 添加元素从而让切片的容量更大的时候，实际上是创建了一个新数组，把原来的切片元素和新添加的元素放到新的数组里，并把这个数组作为新 slice 的底层。如果你添加很多数据到 slice 里，就会反复去创建和复制这些数据，影响性能。所以运行时会分配比你期望的更多的容量到 slice，让复制数据这些操作变得不那么频繁。

虽然原因找下来，感觉这个问题似乎不怎么重要，不过有人如果看到这里，跟我有同样的疑惑，也可以做个参照。

**参考资料：**

- [https://stackoverflow.com/questions/38573983/capacity-of-slices-in-go](https://stackoverflow.com/questions/38573983/capacity-of-slices-in-go)
- [https://stackoverflow.com/questions/38543825/appending-one-element-to-nil-slice-increases-capacity-by-two](https://stackoverflow.com/questions/38543825/appending-one-element-to-nil-slice-increases-capacity-by-two)
- [Go-append使用方法及注意事项](https://blog.csdn.net/a_flying_bird/article/details/54428546)
- [Go语言小知识之append()函数](https://blog.csdn.net/zxhoo/article/details/70159926)
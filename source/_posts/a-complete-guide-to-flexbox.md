---
title: Flexbox 完全指南
date: 2018-07-16 15:32:48
tags: css
---
## 背景知识
Flexbox Layout (Flexible Box) 模式用于提供一个更有效率的布局途径，让容器(container) 中的子元素(items) 在尺寸未知或者动态变化的时候，也能对齐和分配项目之间的空白，因为称作 flex (弹性)。

Flex 布局的主要思想是让容器能够改变里面项目的宽度/高度(或者顺序)，以便用最佳方式填充可用空间(大多用来适应各种不同的设备和屏幕尺寸)。flex 容器能让里面的子元素展示以适应可用空间，或者让它们收缩以防止溢出。

更重要的是，flexbox 的布局方式与常规布局(比如 block 就是基于垂直方向的，而 inline 是基于水平方向的)相比是方向无关的。虽然常规布局对页面来说可以做得很好，但它们缺乏灵活性去适应大型 APP 或复杂的 APP (特别当方向改变，窗口缩放，窗口拉伸，收缩等等)。

**注意：** Flexbox 布局方式适合的是组件化的、小型的布局，如果是大型的布局，[Grid 布局](http://css-tricks.com/snippets/css/complete-guide-grid/)更为适合。

## 基础知识和相关术语

Flexbox 是一个完整的模式，而不是一个单独的属性，所以它包含了整套的属性。这些属性有的是设置到容器上的(父级元素，也就是 flex container)，有的是设置到子元素上的(子元素，也就是 flex items)。

如果说常规的布局是基于 block 和 inline 的流动方向，那么 flex 布局基于的就是 flex 的流动方向。下图指明了 flex 布局的规范。

![specification](https://cdn.css-tricks.com/wp-content/uploads/2011/08/flexbox.png)

通常的，项目元素会基于主轴(从 main-start 到 main-end)或者交叉轴(从 cross-start 到 cross-end)陈列。

- **main axis 主轴** - 主轴是 flex 容器里面的子元素陈列的基础轴，它不一定是水平方向的，它的方向依赖于 `flex-direction` 属性决定
- **main-start | main-end** - flex 子元素会依照 main-start 到 main-end 的方向来陈列
- **main size** - 一个 flex 元素的宽度或高度，取决于主轴方向，这两个值中的一个，就是 main size。flex 元素的 main size 属性是 `width` 或者 `height` 中的一个
- **cross axis** 交叉轴 - 垂直于主轴，它的方向取决于主轴的方向
- **cross-start | cross-end** - 容器的 flex 行里面填充的子元素，也会按照 cross-start 向着 cross-end 的方向摆放
- **cross size** - 一个 flex 子元素的宽度或高度，在交叉轴方向上，两个值中的一个

## 父级容器的属性(flex container)

### display

用来定义一个 flex 容器，inline 级还是 block 级别取决于设置的值。它为 flex 容器的直接子元素定义了 flex 上下文。

```css
.container {
    display: flex; /* or inline-flex */
}
```

### flex-direction

![flex-direction](https://css-tricks.com/wp-content/uploads/2013/04/flex-direction2.svg)

这个属性确定主轴的方向，定义了子元素在父容器中的陈列。Flexbox 是一种单向布局理念，flex 子元素基本都是以水平行或者垂直列的方式呈现。

```css
.container {
    flex-direction: row | row-reverse | column | column-reverse;
}
```

- `row` (默认值)：在 `ltr` 语言中是从左到右，`rtl` 语言中是从右到左
- `row-reverse`： 在 `ltr` 语言中是从右到左，`rtl` 语言中是从左到右
- `column`：跟 `row` 相同，但是是从上到下
- `column-reverse`：跟 `row-reverse` 相同，但是是从下到上

### flex-wrap

![flex-wrap](https://css-tricks.com/wp-content/uploads/2014/05/flex-wrap.svg)

默认的，flex 子元素会试着呈现在一行里面，改变这个值可以让子元素按需折行

```css
.container {
    flex-wrap: nowrap | wrap | wrap-reverse;
}
```

- `nowrap` (默认值)：所有的 flex 子元素都位于一行
- `wrap`：所有的 flex 子元素可以换行陈列于若干行，从上至下
- `wrap-reverse`：所有的 flex 子元素可以换行陈列于若干行，从下至上

### flex-flow (给 flex 容器设置 flex-direction 和 flex-wrap 的缩写)

这只是 `flex-direction` 和 `flex-wrap` 的缩写方法而已，用于同时定义父容器的主轴交叉轴和是否折行，默认值是 `row nowrap`

```css
flex-flow: <'flex-direction'> || <'flex-wrap'>
```

### justify-content

![justify-content](https://cdn.css-tricks.com/wp-content/uploads/2013/04/justify-content-2.svg)

此属性用于定义主轴的对齐方式。它用于分配剩余的空白空间(It helps distribute extra free space left over when either all the flex items on a line are inflexible, or are flexible but have reached their maximum size.)。同时它也能控制子元素在超过一行时的对齐。

```css
.containter {
    justify-content: flex-start | flex-end | center | space-between | space-around | space-evenly;
}
```

- `flex-start` (默认值)：子元素从起始处开始呈现
- `flex-end`：子元素从末尾开始呈现
- `center`：子元素中间对齐
- `space-between`： 子元素均匀分布于行内，首元素位于行起始
- `space-around`：子元素及其周围空间都会均匀分布。值得注意的是，视觉上空白并不均匀，因为所有子元素都有两个边缘。首元素与父容器的边缘有一单位的空白，但与下一个子元素的边缘有两个单位的空白(也就是上一个元素的右空白与下一个元素的左空白叠加在一起)
- `space-evenly`：子元素均匀分布，每两个子元素之间的空白都是相等的

### align-items

![align-items](https://cdn.css-tricks.com/wp-content/uploads/2014/05/align-items.svg)

用来定义 flex 子元素在交叉轴方向上，默认的陈列方式。可以看作是 `justify-content` 在交叉轴上的版本。

```css
.container {
    align-items: flex-start | flex-end | center | baseline | stretch;
}
```

- `flex-start`：从 cross-start 的起始边缘开始呈现
- `flex-end`： 从 cross-end 的结束边缘开始呈现
- `center`：子元素垂直居中于交叉轴
- `baseline`：子元素按它们的 baseline 对齐
- `stretch` (默认值)：拉伸以填满容器(同时受到 min-width / max-width 的影响)

### align-content

![algin-content](https://css-tricks.com/wp-content/uploads/2013/04/align-content.svg)

这个属性用于处理 flex 容器内行与行之间、在交叉轴上有多余空间时的对齐问题，如同 `justify-content` 在主轴上对子元素对齐的处理。

**注意：** 这个属性在 flex 子元素只有一行的情况下是不生效的。

```css
.container {
    align-content: flex-start | flex-end | center | space-between | space-around | stretch;
}
```

- `flex-start`：行从父容器起始处呈现
- `flex-end`：行从父容器结束处呈现
- `center`：行与父容器中间对齐
- `space-between`：行之间均匀分布，第一行处于父容器起始，最后一行处理父容器结束处
- `space-around`：行之间均匀分布，并且每行的空白也是平均的
- `stretch` (默认值)：每行伸展以占据空白区域

## 子元素的属性(flex items)

### order

![order](https://css-tricks.com/wp-content/uploads/2013/04/order-2.svg)

默认的，flex 子元素是按源码里的顺序来排列的。然而 `order` 属性可以控制它们在父容器中出现的顺序。

```css
.item {
    order: <integer>; /* default is 0 */
}
```

### flex-grow

![flex-grow](https://css-tricks.com/wp-content/uploads/2014/05/flex-grow.svg)

这个属性定义的是 flex 子元素按需增长的能力。它接受的值是无单的数字作为比例，它规定了子元素占据 flex 父容器内可用空间的量。

如果所有的子元素都把 `flex-grow` 设置为 1，那么父容器中剩余的空间将会平分给每一个子元素。如果其中一个子元素的值设置为 2，那么它将占据其他子元素 2 倍的空间 (或者试图占据 2 倍)。

```css
.item {
    flex-grow: <number>; /* default 0 */
}
```

负数的值是非法的。

### flex-shrink

这个属性定义的是 flex 子元素按需收缩的能力。

```css
.item {
    flex-shrink: <number>; /* default 1 */
}
```

负数的值是非法的。

### flex-basis

它定义的是子元素在分配到剩余空间之前的元素的默认尺寸。它的值可以是长度 (例如：20%，5rem 等等) 或者是关键字。`auto` 这个关键字意义是“参照宽度 width 或者高度 height 属性” (这是由 `main-size` 关键字暂代实现，直到被弃用为止)。`content` 这个关键字意义是“尺寸由子元素的内容来决定” - 这个关键字并没有被很好的支持，所以也不好测试，更不知道是 `max-content`、`min-content` 或 `fit-content` 中的哪个起的作用。

```css
.item {
    flex-basis: <length> | auto; /* default auto */
}
```

如果被设置为 0，那么内容周围的多余空白就没被考虑进去，如果设置为 `auto`，那么多余空白的分布，取决于它的 `flex-grow` 值。见下图：

![flex-basis](https://www.w3.org/TR/css-flexbox-1/images/rel-vs-abs-flex.svg)

### flex

这个属性是 `flex-grow`、`flex-shrink` 和 `flex-basis` 三个属性的简写方式，其中第二和第三个参数(也就是 `flex-shrink` 和 `flex-basis`)是可选的。默认值是 `0 1 auto`。

```css
.item {
    flex: none | [ <'flex-grow'> <'flex-shrink'>? || <'flex-basis'> ]
}
```

**强烈建议使用这个缩写的属性** 而不是单独设置各种属性的值。因为缩写方法会智能地设置其他值。

### align-self

![align-self](https://css-tricks.com/wp-content/uploads/2014/05/align-self.svg)

这个属性允许单个的 flex 子元素的默认的对齐(或者被 `align-item` 声明过)被覆盖。

参看 `align-items` 解释，看可设置的值。

```css
.item {
    align-self: auto | flex-start | flex-end | center | baseline | stretch;
}
```

注意：`float`、`clear` 和 `vertical-align` 对 flex 子元素无影响。

## 参考资料

[A Complete Guide to Flexbox - css-tricks.com](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)
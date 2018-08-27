---
title: 一些数组方法的备忘
date: 2018-08-27 15:09:38
tags: javascript
---
## forEach

`forEach()` 方法对数据的每一个元素执行一次提供的函数

### 语法

```javascript
array.forEach(callback (currentValue, index, array) {}[, thisArg]);
```

- **callback** - 为数组中每个元素执行的函数，接收三个参数
- **currentVale** - `callback` 的第一个参数，当前正处理的元素
- **index** - 可选参数，`callback` 的第二个参数，当前正处理的元素的索引
- **array** - 可选参数，`callback` 的第三个参数，是正在处理的数组
- **thisArg** - 可选参数，当执行回调函数时用作 this 的值（参考对象）

### 返回值

`undefined`

### 注意

没有办法中止或者跳出 forEach 循环，除非抛出一个错误。

## filter

`filter()` 方法**创建一个新数组**，包含通过所提供的函数实现的测试的所有元素。

### 语法

```javascript
var newArray = array.filter(callback (currentValue[, index[, array]]) {}[, thisArg]);
```

- **callback** - 用来测试数组每个元素的函数，调用时使用参数 `(currentValue, index, array)`，返回 `true` 表示保留该元素（通过了测试），`false` 则不保留
- **currentValue** - `callback` 的第一个参数，当前正在处理的元素
- **index** - 可选参数，`callback` 的第二个参数，当前正在处理的元素的索引值
- **array** - 可选参数，`callback` 的第三个参数，调用 `filter` 的数组
- **thisArg** - 可选参数，当执行回调函数时用作 this 的值（参考对象）

### 返回值

一个**新的**通过测试的元素集合的**数组**，如果没有元素通过测试，就返回**空数组**。

## map

`map()` **创建一个新数组**，新数组中的每个元素，是原数组里每个元素，调用所提供的函数后返回的结果。

### 语法

```javascript
var newArray = array.map(callback (currentValue[, index[, array]]) {}[, thisArg]);
```

- **callback** - 生成新数组元素的函数
- **currentVale** - `callback` 的第一个参数，数组中当前正在处理的元素
- **index** - 可选参数，`callback` 的第二个参数，数组中当前正在处理的元素的索引值
- **array** - 可选参数，`callback` 的第三个参数，调用 `map` 的数组
- **thisArg** - 可选参数，执行回调函数时用作 this 的值（参考对象）

### 返回值

一个**新数组**，每个元素都是回调函数的结果。

## reduce

`reduce()` 对累加器和数组中的每个元素（从左到右）应用一个函数，将其减少为单个值。

### 语法

```javascript
array.reduce(callback (accumulator, currentValue[, currentIndex[, array]]) {}[, initialValue]);
```

- **callback** - 执行数组中每个值的函数
- **accumulator** - `callback` 的第一个参数，累加器累加回调的返回值，它是上一次调用回调时返回的累积值，或 `initialValue` 
- **currentValue** - `callback` 的第二个参数，数组中当前正在处理的元素
- **currentIndex** - 可选参数，`callback` 的第三个参数，数组中当前正在处理的元素的索引。如果提供了 `initialValue`，则索引号为 0，否则索引为 1
- **array** - 可选参数，`callback` 的第四个参数，调用 `reduce` 的数组
- **initialValue** - 可选参数，用作第一个调用 `callback` 的第一个参数的值。如果没有提供初始值，则将使用数组中的第一个元素。在没有初始值的空数上调用 `reduce` 会报错

### 返回值

函数累计处理的结果

### 注意

如果没有提供 `initialValue`，`reduce` 会从索引 1 的地方开始执行 callback 方法，跳过第一个索引。如果提供 `initialValue`，从索引 0 开始。

## reduceRight

`reduceRight` 总体上与 `reduce` 相同，区别在于，使用数组中的值，从右向左传给 `callback`

## every

`every()` 测试数组中所有元素是否都通过了指定函数的测试。

### 语法

```javascript
array.every(callback (currentValue[, index[, array]]) {}[, thisArg]);
```

- **callback** - 用来测试每个元素的函数
- **currentValue** - `callback` 的第一个参数，当前处理的元素
- **index** - 可选参数，`callback` 的第二个参数，当前处理的元素的索引值
- **array** - 可选参数，`callback` 的第三个参数，调用 `every` 的数组
- **thisArg** - 可选参数，执行 `callback` 时使用的 this 值

### 返回值

如果每个元素都通过了测试函数，则返回 `true`，否则返回 `false`。

### 注意

在空数组上调用 `every` 方法，无论测试函数是什么，都会返回 `true`。

## some

`some()` 测试数组中某些元素是否通过指定函数的测试。

### 语法

```javascript
array.some(callback (currentValue[, index[, array]]) {}[, thisArg]);
```

- **callback** - 测试函数
- **currentValue** - `callback` 的第一个参数，当前正在处理的元素
- **index** - 可选参数，`callback` 的第二个参数，当前正在处理的元素的索引值
- **array** - 可选参数，`callback` 的第三个参数，调用 `some` 的数组
- **thisArg** - 可选参数，执行 `callback` 时使用的 this 值

### 返回值

如果任意一个元素通过了测试函数，就返回 `true`，否则返回 `false`。

### 注意

在空数组上调用 `some` 方法，无论测试函数是什么，都会返回 `false`。

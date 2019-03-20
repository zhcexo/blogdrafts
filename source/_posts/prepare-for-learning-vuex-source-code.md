---
title: 学习Vuex源码前的准备——变量的解构赋值
date: 2019-03-20 13:55:39
tags: ['javascript', 'vuex']
---
试了一下 Vue，然后也学习了一下 Vuex，发现 Vuex 的源代码好像不长，就想看看试试。不过由于是用 ES6+ 写的，与传统的写法有点不同，所以如果没有看过 ES6+，那么有必要了解一些必备的知识。

## 数组的扩展运算符

扩展运算符是三个点 `...`，用于将一个**数组**转为用逗号分隔的**参数序列**，它主要用于**函数调用**。

要注意上面提到的，是转化为**参数序列**，并不是单单用于数组拆解的，所以如果你打开 DevTools 的 console，在里面输入 `...[1, 2, 3]`，那么你不会得到 `1,2,3`，而是会得到一个错误：`Uncaught SyntaxError: Unexpected token ...`。

```javascript
console.log(...[1, 2, 3])
// 1 2 3

console.log(1, ...[2, 3, 4], 5)
// 1 2 3 4 5

[...document.querySelectorAll('div')]
// [<div>, <div>, <div>]

{...document.querySelectorAll('div')}
// {0: <div>, 1: <div>, 2: <div>}

function push(array, items) {
  array.push(...items);
  return array;
}

function add(x, y) {
  return x + y;
}

const numbers = [4, 38];
add(...numbers);    // 42
```

注意上面的例子，把解构出来的值，放到数组 `[]` 里面，那么得到的是个数组；而如果把解构出来的值，放到对象 `{}` 里面，那么会得到一个以 `index` 为 `key`，元素为 `value` 的对象，这个对象是没有 `length` 属性的。

扩展运算符后面可以放置**表达式**。如果扩展运算符后面是一个空数组，则不产生任何效果。另外，扩展运算符如果放在括号中，JavaScript 引擎会认为这是函数调用，**如果这时不是函数调用，就会报错**。

```javascript
const arr = [
  ...(x > 0 ? ['a'] : []),
  'b'
];

[...[], 1]
// 1

(...[1, 2, 3])
// Uncaught SyntaxError: Unexprected token ...

console.log((...[1, 2, 3]))
// Uncaught SyntaxError: Unexprected token ...

console.log(...[1, 2, 3])
// 1 2 3
```

### 替代函数的 apply 方法

由于扩展运算符可以展开数组，所以不再需要 `apply` 方法，将数组转为函数的参数了。

```javascript
// 旧的写法
function f(x, y, z) {
  // ...
}
var args = [0, 1, 2];
f.apply(null, args);

// ES6 的写法
function f(x, y, z) {
  // ...
}
let args = [0, 1, 2];
f(...args);
```

下面是用扩展运算符取代 `apply` 方法的一个实际例子，应用 `Math.max` 方法，简化求出一个数最大元素的写法。

```javascript
// 旧的写法
Math.max.apply(null, [14, 3, 77]);

// ES6 的写法
Math.max(...[14, 3, 77]);

// 等同于
Math.max(14, 3, 77);
```

### 扩展运算符的应用

#### 复制数组

数组是复合数据类型，直接复制的话，只是复制了指向底层数据结构的指针，而不是克隆一个全新的数组。下面的代码中，`a2` 并不是 `a1` 的克隆，而是指向同一份数据的另一个指针。修改 `a2` 会直接导致 `a1` 的变化。

```javascript
const a1 = [1, 2];
const a2 = a1;

a2[0] = 2;
console.log(a1);
// [2, 2]
```

在 ES5 中，一般使用变通的方法来复制数组

```javascript
const a1 = [1, 2];
const a2 = a1.concat();

a2[0] = 2;
console.log(a1);
// [1, 2]
```

用扩展运算符来写的话，就会比较简便，两种写法里，`a2` 都是 `a1` 的克隆：

```javascript
const a1 = [1, 2];
const a2 = [...a1];
// 或者
const [...a2] = a1;
```

#### 合并数组

扩展运算符提供了数组合并的新写法：

```javascript
const a1 = ['a', 'b'];
const a2 = ['c'];
const a3 = ['d', 'e'];

// 旧的写法
a1.concat(a2, a3);
// ['a', 'b', 'c', 'd', 'e']

// ES6 的写法
[...a1, ...a2, ...a3]
// ['a', 'b', 'c', 'd', 'e']
```

如果数据里面的元素，是复合的数据类型，无论是 `concat` 还是用解构运算符，都是**浅复制**，也就是，新数组里面的成员，依然是对原数组成员的引用（指针），如果修改了原数组成员，那么新数组中成员也会变化。

```javascript
const a1 = [{foo: 1}];
const a2 = [{bar: 2}];

const a3 = a1.concat(a2);
const a4 = [...a1, ...a2];

a3[0] === a1[0]     // true
a4[0] === a1[0]     // true
```

#### 与解构赋值结合

扩展运算符可以与解构赋值结合起来，用于生成数组。如果将扩展运算符用于数组赋值，**只能放在参数的最后一位**，否则会报错。

```javascript
const [first, ...rest] = [1, 2, 3, 4, 5];
first // 1
rest  // [2, 3, 4, 5]

const [first, ...rest] = [];
first // undefined
rest  // []

const [first, ...rest] = ['foo'];
first // "foo"
rest  // []

const [...butLast, last] = [1, 2, 3, 4, 5];
// Uncaught SyntaxError: Rest element must be last element

const [first, ...middle, last] = [1, 2, 3, 4, 5]
// Uncaught SyntaxError: Rest element must be last element
```

#### 字符串

扩展运算符可以将字符串转化为真正的数组，**而且能正确识别四个字节的 Unicode 字符**。

```javascript
[...'hello']
// ['h', 'e', 'l', 'l', 'o']

'\uD83D\uDE80'.length   // 4
// \uD83D\uDE80 是一个四字节的 Unicode 字符，所以 length 应该是 1
// 但是直接用 length 被识别成了 2 个字符

[...'\uD83D\uDE80'].length  // 1
// 用扩展运算符就能正确识别
```
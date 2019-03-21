---
title: 学习Vuex源码前的准备——变量的解构赋值
date: 2019-03-21 10:05:56
tags: javascript
---
继续第二章，变量的解构赋值。

## 什么是解构赋值

ES6 允许按照一定的**模式**，**从数组和对象中提取值**，这种方式被称为解构（Destructuring）。

## 数组的解构赋值

### 基本用法

以前为变量赋值，只能直接指定值，例如：

```javascript
let a = 1;
let b = 2;
let c = 3;
```

但是解构赋值允许这样写：

```javascript
let [a, b, c] = [1, 2, 3];
```

意义就跟上面一样了，为 `a`，`b`，`c` 分别赋值。

本质上，这种写法属于**“模式匹配”**，只要等号两边的模式相同，左边的变量就会被赋予相应的值。以下是更多例子：

```javascript
let [foo, [[bar], baz]] = [1, [[2], 3]];
foo // 1
bar // 2
baz // 3

let [ , , third] = ['foo', 'bar', 'baz'];
third   // baz

let [x, , y] = [1, 2, 3];
x   // 1
y   // 3

let [head, ...tail] = [1, 2, 3, 4];
head    // 1
tail    // [2, 3, 4]

let [x, y, ...z] = ['a'];
x   // 'a'
y   // undefined
z   // []
```

如果解构不成功，变量的值就等于 `undefined`。

另一种情况是不完全解构，即等号左边的模式，只匹一部分的等号右边的数组。这种情况下，解构依然可以成功。

```javascript
let [x, y] = [1, 2, 3];
x   // 1
y   // 2

let [a, [b], d] = [1, [2, 3], 4];
a   // 1
b   // 2
d   // 4
```

如果等号的右边不是数组（或者严格的说，不是可遍历的结构），那么将会报错。

### 默认值

解构赋值允许指定默认值。但要注意的是，ES6 内部使用严格相等运算符号（`===`），判断一个位置是否有值。所以，只有当一个数组成员严格等于 `undefined`，默认值才会生效。

```javascript
let [foo = true] = [];
foo // true

let [x, y = 'b'] = ['a']
x   // 'a'
y   // 'b'

let [x, y = 'b'] = ['a', undefined];
x   // 'a'
y   // 'b'

let [x = 1] = [undefined];
x   // 1

let [x = 1] = [null];
x   // null
```

如果默认值是一个表达式，那么这个表达式是**惰性求值**的，即只有在用到的时候，才会求值。

```javascript
function f() {
  console.log('aaa');
}

let [x = f()] = [1];
// f() 不会执行，所以 x 能获得值，是 1、

let [x = f()] = [];
// 解构的右边是 undefined，x 取不到值，那么默认赋值开始，f() 执行
// 此时 x 的值为 f() 的返回值，此例中是 undefined （无返回值）
```

## 对象的解构赋值

对象的解构赋值和数组有一个重要的不同，数组的元素是按次序排列的，变量的取值由它的位置决定；而对象的属性没有次序，**变量必须与属性同名**，才能取到正确的值。

```javascript
let {foo, bar} = {foo: 'aaa', bar: 'bbb'};
foo // 'aaa'
bar // 'bbb'

let {bar, foo} = {foo: 'aaa', bar: 'bbb'};
foo // 'aaa'
bar // 'bbb'

let {baz} = {foo: 'aaa', bar: 'bbb'};
baz // undefined
```

如果变量名与属性名不一致，必须写成下面这样：

```javascript
let {foo: baz} = {foo: 'aaa', bar: 'bbb'};
foo // Uncaught ReferenceError: foo is not defined
baz // 'aaa'

let obj = {first: 'hello', last: 'world'};
let {first: f, last: l} = obj;
f   // 'hello'
l   // 'world'
first   // Uncaught ReferenceError: first is not defined
last    // Uncaught ReferenceError: last is not defined
```

实际上说明，对像的解构赋值是下面形式的简写：

```javascript
let {foo: foo, bar: bar} = {foo: 'aaa', bar: 'bbb'};
```

也就是说，对象的解构赋值的内部机制，是先找到同名属性，然后再赋给对应的变量。**真正被赋值的是后者，而不是前者。**

与数组一样，解构也可以用于嵌套结构的对象。

```javascript
let obj = {
  p: [
    'Hello',
    {
      y: 'World'
    }
  ]
};

let {p: [x, {y}]} = obj;
x   // 'Hello'
y   // 'World'
```

在上面的例子中，`p` 是模式，不是变量，所以 `p` 不会被赋值，如果 `p` 也要作为变量赋值，要写成下面这样：

```javascript
// 续上
let {p, p: [x, {y}]} = obj;
p   // ['Hello', {y: 'World'}]
x   // 'Hello'
y   // 'World'

// 另一个例子
const node = {
  loc: {
    start: {
      line: 1,
      column: 5
    }
  }
};

let {loc, loc: {start}, loc: {start: {line, column}}} = node
loc     // Object {start: {line: 1, column: 5}}
start   // Object {line: 1, column: 5}
line    // 1
column  // 5
```

### 默认值

同数组解构的默认值一样，**对象的属性值严格等于 `undefined` 时默认值才会生效**。如果解构失败，变量的值等于 `undefined`。

```javascript
var {x = 3} = {};
x   // 3

var {x, y = 5} = {x: 1};
x   // 1
y   // 5

var {x: y = 3} = {};
x   // error，要记住，x 是模式，不赋值
y   // 3

var {x: y = 3} = {x: 5};
y   // 5

var {message: msg = 'Something went wrong'} = {};
msg // 'Something went wrong'

// 对象的属性值严格等于 undefined 时默认值才会生效
var {x = 3} = {x: undefined};
x   // 3

var {x = 3} = {x: null};
x   // null

// 解构失败的例子
let {foo} = {bar: 'baz'};
foo // undefined
```

如果解构模式是嵌套的对象，而且子对象所在的父属性不存在，那么将会报错。如果要将一个已经声明的变量用于解构赋值，也要小心。

```javascript
let {foo: {bar}} = {baz: 'baz'};
// Uncaught TypeError: Cannot destructure property `bar` of 'undefined' or 'null'.

// 上面的解构相当于下面的代码，所以才会报错
let _tmp = {baz: 'baz'};
_tmp.foo.bar
// Uncaught TypeError: Cannot read property 'bar' of undefined

// 错误的写法
let x;
{x} = {x: 1};
// Uncaught SyntaxError: Unexpected token =
```

上面的写法会报错，因为 JS 引擎会将 `{x}` 解释成一个代码块，从而发生语法错误。**只有不将花括号写在行首，避免 JS 将它解释成代码块，才能解决问题**。

```javascript
let x;
({x} = {x: 1});
```

解构赋值允许等号左边的模式之中，不放置任何变量名，因为可以写出非常古怪的表达式，它们没有意义，但是语法合法，可以执行。

```javascript
({} = [true, false]);
({} = 'abc');
({} = []);
```

对象的解构赋值，可以很方便的将现有对象的方法，赋值到某个变量。

```javascript
let {log, sin, cos} = Math;
```

由于数组的本质也是对象，因此可以对数组进行对象的解构。

```javascript
let arr = [1, 2, 3];
let {0: first, [arr.length -1]: last} = arr;
first   // 1
last    // 3
```

## 字符串的解构赋值

对字符串进行解构赋值时，字符串被转换成了一个类似数组的对象。

```javascript
const [a, b, c, d, e] = 'hello';
a  // 'h'
b  // 'e'
c  // 'l'
d  // 'l'
e  // 'o'
```

因为字符串还有个 `length` 属性，所以这个属性也可被解构。

```javascript
let {length: len} = 'hello';
len  // 5
```

## 数值和布尔值的解构赋值

如果等号右边是数值和布尔值，则会先转为对象。

```javascript
let {toString: s} = 123;
s === Number.prototype.toString     // true

let {toString: s} = true;
s === Boolean.prototype.toString    // true
```

解构赋值的规则是，只要等号右边的值不是对象或数组，就先将其转为对象。由于 `undefined` 和 `null` 无法转换成对象，所以对它们进行解构的时候，会报错。

```javascript
let {prop: x} = undefined;
let {prop: y} = null;

// Uncaught TypeError: Cannot destructure property `prop` of 'undefined' or 'null'.
```

## 函数参数的解构赋值

函数的参数也可以使用解构赋值。

```javascript
function add([x, y]) {
  return x + y;
}
add([1, 2]);
// 3

[[1, 2], [3, 4]].map(([a, b]) => a + b);
//[3, 7]

// 使用默认值的情况
function move({x = 0, y = 0} = {}) {
  return [x, y];
}
move({x: 3, y: 8});     // [3, 8]
move({x: 3});           // [3, 0]
move({});               // [0, 0]
move();                 // [0, 0]
```

不过，如果用下面的写法，会得到不一样的结果：

```javascript
function move({x, y} = {x: 0, y: 0}) {
    return [x, y];
}
move({x: 3, y: 8});     // [3, 8]
move({x: 3});           // [3, undefined]
move({});               // [undefined, undefined]
move();                 // [0, 0]
```

上面这段代码是为函数 `move` 的参数指定默认值，而不是为 `x` 和 `y` 变量指定默认值，所以会得到不同结果。

`undefined` 就会触发函数参数的默认值。

```javascript
[1, undefined, 3].map((x = 'yes') => x);
// [1, 'yes', 3]
```

## 圆括号的问题

解构的过程中，如果模式中出现了圆括号，可能会引起处理上的歧义。ES6 的规则是，只要有可能导致解构的歧义，就不得使用圆括号。

### 不能使用圆括号的情况

#### 变量声明语句

```javascript
// 全部报错
let [(a)] = [1];

let {x: (c)} = {};
let ({x: c}) = {};
let {(x: c)} = {};
let {(x): c} = {};

let {o: ({p: p})} = {o: {p: 2}};
```

#### 函数参数

函数参数也属于变量声明，因此不能带有圆括号。

```javascript
// 全部报错
function f([(z)]) { return z; }
function f([z, (x)]) { return x; }
```

#### 赋值语句的模式

```javascript
// 全部报错
({p: a}) = {p: 42};
([a]) = [5];
```

### 可以使用圆括号的情况

可以使用圆括号的情况只有一种：赋值语句的非模式部分，可以使用圆括号。

```javascript
// 以下均正确
[(b)] = [3];                // 模式取的是数组的第一个成员，跟圆括号无关
({p: (d)} = {});            // 模式是 p，不是 d
[(parseInt.prop)] = [3];    // 跟第一行一样，取的是数组的第一个成员，跟圆括号无关
```

除了上面注释里写明的原因，上面三行都能正确执行的原因，是**它们都是赋值语句**，而**不是声明语句**。

## 用途

### 交换变量的值

```javascript
let x = 1;
let y = 2;

[x, y] = [y, x];
```

### 从函数返回多个值

函数只有一个返回值，如果要返回多个值，只能将它们放在数组或者对象里返回。然后用解构赋值取出它们。

```javascript
// 返回一个数组
function example() {
  return [1, 2, 3];
}
let [a, b, c] = example();

// 返回一个对象
function example() {
  return {
    foo: 1,
    bar: 2
  }
}
let {foo, bar} = example();
```

### 函数参数的定义

解构赋值可以方便的将一组参数与变量名对应起来。

```javascript
// 参数是一组有序的值
function f([x, y, z]) { ... }
f([1, 2, 3]);

// 参数是一组无次序的值
function f({x, y, z}) { ... }
f({z: 3, y: 2, x: 1});
```

### 提取 JSON 数据

```javascript
let jsonData = {
  id: 42,
  status: 'OK',
  data: [867, 5309]
};
let {id, status, data: number} = jsonData;
console.log(id, status, number);
// 42, 'OK', [867, 5309]
```

### 函数参数默认值

```javascript
jQuery.ajax = function (url, {
  async = true,
  beforeSend = function () {},
  cache = true,
  complete = function () {},
  crossDomain = false,
  global = true
});
```

## 在 Vuex 中的应用

创建带命名空间的辅助函数：

```javascript
import {createNamespacedHelpers} from 'vuex';

const {
  mapGetters: postsGetters,
  mapActions: postsActions
} = createNamespacedHelpers('posts');
```

然后添加组件定义：

```javascript
export default {
  computed: {
    ...postsGetters([
      'draft',
      'currentPost'
    ]),
    cssClass() {
      return [
        'blog-content',
        {
          'has-content': this.currentPost
        }
      ]
    }
  }
}
```

重命名带命名空间的辅助函数是个很好的实践，因为未来可能还会为其他模块添加辅助函数。例如，如果不这么做，可能最后会有两个 `mapGetters`，而这是不可行的。这里将 `mapGetters` 重命名为 `postsGetters`，同时将 `mapActions` 重命名为 `postsActions`。

## 参考资料

[变量的解构赋值](http://es6.ruanyifeng.com/#docs/destructuring)

本文基本是上面链接的笔记和再整理。
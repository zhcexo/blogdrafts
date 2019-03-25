---
title: 学习Vuex源码前的准备——模块及加载实现
date: 2019-03-21 17:12:23
tags: javascript
---
## 概述

在 ES6 之前，社区制定了一些模块加载方案，最主要的有 CommonJS 和 AMD 两种。前者用于服务器，后者用于浏览器。ES6 在语言标准层面上，实现了模块功能，而且实现得相当简单，完全可以取代 CommonJS 和 AMD 规范，成为浏览器和服务器通用模块解决方案。

ES6 模块的设计思想是尽量的静态化，使得编译时就能确定模块的依赖关系，以及输入和输出的变量。CommonJS 和 AMD 模块都只能在运行时确定这些东西。

**ES6 模块不是对象，而是通过 `export` 命令显式指定输出的代码，再通过 `import` 命令输入。**

由于 ES6 模块是编译时加载，使用静态分析成为可能。

## 严格模式

ES6 的模块**自动采用严格模式**，不管有没有在模块的头部加上 `"use strict"`。

严格模式的限制：

- 变量必须声明后再使用
- 函数的参数不能有同名属性，否则报错
- 不能使用 `with` 语句
- 不能对只读属性赋值，否则报错
- 不能使用前缀 0 的八进制数，否则报错
- 不能删除变量 `delete prop`，会报错，只能删除属性 `delete global[prop]`
- `eval` 不会在它的外层作用域引入变量
- `eval` 和 `arguments` 不能被重新赋值
- `arguments` 不会自动反映函数参数的变化
- 不能使用 `arguments.callee`
- 不能使用 `arguments.caller`
- 禁止 `this` 指向全局变量
- 不能使用 `fn.caller` 和 `fn.arguments` 获取函数调用的堆栈
- 增加了保留字（比如 `protected`、`static`和`interface`）

其中，尤其需要注意 `this` 的限制。ES6 模块之中，顶层的 `this` 指向 `undefined`，即不应该在顶层代码使用 `this`。

## export 命令

`export` 命令用于规定模块的对外接口。**一个模块就是一个独立的文件**，该文件内部的所有变量，外部无法获取 。如果希望外部能够读取模块内部的某个变量，必须使用 `export` 关键字输出该变量。例如：

```javascript
export var firstName = 'Michael';
export var lastName = 'Jackson';
export var year = 1958;

// or

var firstName = 'Michael';
var lastName = 'Jackson';
var year = 1958;
export {firstName, lastName, year};

// 用 as 关键字重命名
function v1() { ... }
function v2() { ... }
export {
    v1 as streamV1,
    v2 as streamV2,
    v2 as streamLatestVersion
}
```

特别注意： `export` 命令规定的是对外的接口，必须与模块内部的变量建立一一对应关系。

```javascript
// 报错
export 1;

// 报错
var m = 1;
export m;
```

上面的代码报错是因为没有提供对外的接口。第一种直接输出 1，第二种通过变量 `m` 还是直接输出的 1。`1` 只是一个值，不是接口。正确的写法如下：

```javascript
// 写法一
export var m = 1;

// 写法二
var m = 1;
export {m};

// 写法三
var n = 1;
export {n as m};
```

上面的三种写法都规定了对外接口 `m`，其他的脚本可以通过这个接口取到值 `1`，所以三种写法都对。**它们的实质是，在接口与模块内部变量之间，建立了一一对应的关系。**

`function` 和 `class` 的输出，也必须遵守这样的写法：

```javascript
// 报错
function f() {}
export f;

// 正确
export function f() {}

// 正确
function f() {}
export {f};
```

另外，`export` 语句输出的接口，与其对应的值是动态绑定关系，即通过该接口，可以取到模块内部实时的值。例如下面的代码，输出的变量是 `foo`，值为 `bar`，但是 500ms 之后变成了 `baz`。

```javascript
export var foo = 'bar';
setTimeout(() => foo = 'baz', 500);
```

## import 命令

使用 `export` 命令定义了模块的对外接口以后，其他 JS 文件就可以通过 `import` 命令加载这个模块。

```javascript
// main.js
import {firstName, lastName, year} from './profile.js';
// or 重命名
import {lastName as surname} from './profile.js';

function setName(element) {
  element.textContext = firstName + '' + lastName;
}
```

`import` 命令输入的变量都是只读的，因为它的本质是输入接口。也就是说，不允许加载模块的脚本里，改写接口。但如果加载的变量是个对象，改成对象的属性是允许的。例如：

```javascript
import {a} from './xxx.js';

a = {};             // 报错

a.foo = 'hello';    // 正确
```

注意，`import` 命令具有提升效果，会提升到整个模块的头部，首先执行。

```javascript
foo();

import {foo} from 'my_module';
```

上面的代码不会报错，因为 `import` 的执行早于 `foo` 的调用。这种行为的本质是，`import` 命令是编译阶段执行的，在代码运行之前。

也正因为 `import` 是静态执行，所以不能使用表达式和变量——这些只有在运行时才能得到结果的语法结构。

```javascript
// 报错
import {'f' + 'oo'} from 'my_module';

// 报错
let module = 'my_module';
import {foo} from module;

// 报错
if (x === 1) {
  import {foo} from 'module1';
} else {
  import {foo} from 'module2';
}
```

上面三种写法都会报错，因为它们用到了表达式、变量和 `if` 结构。在静态分析阶段，这些语法都是没法得到值的。

最后，**`import` 语句会执行所加载的模块**，因此可以有下面的写法：

```javascript
import 'lodash';
```

上面代码仅仅执行 `lodash` 模块，但是不输入任何值。

如果多次重复执行同一句 `import` 语句，那么只会执行一次，而不会执行多次。

```javascript
// 例 1
import 'lodash';
import 'lodash';

// 例 2
import {foo} from 'my_module';
import {bar} from 'my_module';
// 等同于
import {foo, bar} from 'my_module';
```

目前通过 Babel 转码，CommonJS 的 `require` 和 ES6 模块的 `import` 命令可以写在同一个模块里，**但最好不要这样做**。因为 `import` 在**静态解析阶段**执行，所以**它是一个模块之中最早执行的**。

下面的代码可能不会得到预期结果：

```javascript
require('core-js/modules/es6.symbol');
require('core-js/modules/es6.promise');
import React from 'react';
```

## 模块的整体加载

除了指定加载某个输出值，还可以整体加载，即用星号（`*`）指定一个对象，所有输出值都加载在这个对象上面。

```javascript
// circle.js
export function area(radius) {
  return Math.PI * radius * radius;
}

export function circumference(radius) {
  return 2 * Math.PI * radius;
}
```

然后加载 `circle.js` 这个模块

```javascript
import {area, circumference} from './circle.js';
console.log('圆面积：' + area(4));
console.log('圆周长：' + circumference(4));

// or

import * as circle from './circle.js';
console.log('圆面积：' + circle.area(4));
console.log('圆周长：' + circle.circumference(4));
```

注意，模块整体加载所在的那个对象（上例是 `circle`），应该是可以静态分析的，所以不允许运行时改变，以下会报错。

```javascript
import * as circle from './circle.js';

// 下面两行都是不允许的
circle.foo = 'hello';
circle.area = function () {};
```

## `export default` 命令

从上面的例子可以看出，使用 `import` 命令时，需要知道所要加载的变量名或者函数名，否则无法加载。为了方便，可以用 `export default` 命令，为模块指定默认输出。

本质上，`export default` 就是输出一个叫 `default` 的变量或方法，然后系统允许你给它取任意名字。但它后面不能跟变量声明语句。

```javascript
// 第一组----------------------------
export default function foo() {}
// 或者写成
function foo() {}
export default foo;

// 第二组----------------------------
export default function crc32() {}
import crc32 from 'crc32';

// 第三组----------------------------
export function crc32() {};
import {crc32} from 'crc32';

// 第四组----------------------------
export var a = 1;
// 正确

var a = 1;
export default a;
// 正确

export default var a = 1;
// 错误

// 第五组---------------------------
function add() {}
export {add as default};
// 等同于 export default add
import {default as foo} from 'modules';
// 等同于 import foo from 'modules'
```

上面的第一组，`foo` 函数的函数名 `foo`，**在模块外面是无效的**，**加载的时候视同匿名函数**。

上面的第二组，使用 `export default` 时，对应的 `import` 语句不需要使用大括号。

上面的第三组，不使用 `export default`，对应的 `import` 语句需要使用大括号。

上面的第四组，不能在 `export default` 后面使用变量声明语句。

上面的第五组，体现 `export default` 就是输出一个叫 `default` 的变量。

```javascript
// 正确
export default 42;

// 报错
export 42;
```

上面的代码，后一句报错是因为没有指定对外的接口，而前一句的对外接口是 `default`。

输出一个类：

```javascript
// MyClass.js
export default class { ... };

// main.js
import MyClass from 'MyClass';
let o = new MyClass();
```

## `export` 与 `import` 的复合写法

如果在一个模块之中，先输入后输出同一模块，`import` 语句可以与 `export` 语句写在一起。

```javascript
export {foo, bar} from 'my_module';

// 可以简单理解为
import {foo, bar} from 'my_module';
export {foo, bar};
```

上面的代码中，`export` 和 `import` 语句可以结合在一起，写成一行。但需要注意的是，写成一行以后，`foo` 和 `bar` 实际上**并没有被导入当前模块**，只是相当于对外转发了两个接口，导致**当前模块不能直接使用 `foo` 和 `bar`**。

## 浏览器中的模块加载

### 浏览器中加载

浏览器中加载脚本，一般是用下面的代码：

```html
<script src="path/to/myModule.js" defer></script>
<script src="path/to/myModule.js" async></script>
```

`defer` 与 `async` 的区别是：`defer` 要等整个页面在内存中正常渲染结束（DOM结构完全生成，以及其他脚本执行完成），才会执行；`async` 是一旦下载完，渲染引擎就会中断渲染，执行这个脚本后，再继续渲染。一句话，`defer` 是**渲染完再执行**，`async` 是**下载完就执行**。另外，如果有多个 `defer` 脚本，会按照它们在页面出现的顺序加载，而多个 `async` 脚本是不保证加载顺序的。

### 加载规则

```html
<script type="module" src="path/to/module.js"></script>
```

加上 `type="module"` 之后，浏览器就知道这是一个 ES6 模块。浏览器对带有 `type="module"` 的 `<script>`，都是异步加载，不会造成浏览器堵塞，等整个页面渲染完后，再执行脚本，相当于打开了 `defer` 属性。

如果网页中有多个 `<script type="module">`，它们会按照在页面出现的顺序依次执行。

但如果此时 `<script>` 的 `async` 属性也打开了，这时只要加载完成，渲染引擎就会中断渲染立即执行脚本，执行完后再恢复渲染。一旦用了 `async` 属性，`<script>` 就不会按照在页面出现的顺序执行，而是只要该模块加载完成，就执行该模块。

## ES6 与 CommonJS 模块的差异

两个重大差异：

- CommonJS 的模块输出的是一个**值的拷贝**，ES6 模块输出的是**值的引用**。
- CommonJS 的模块是**运行时加载**，ES6 的模块是**编译时输出接口**。

另一个差异是因为 CommonJS 加载的是一个对象（即 `module.exports` 属性），该对象只有在**脚本运行完**才会生成。而 ES6 模块不是对象，它的对外接口只是一种静态定义，在代码**静态解析阶段就会生成**。

## 参考资料

[Module 的加载实现](http://es6.ruanyifeng.com/#docs/module-loader)

本文基本是上面链接的笔记和再整理。

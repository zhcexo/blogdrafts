---
title: 学习Vuex源码前的准备——类 Class
date: 2019-03-25 17:08:17
tags: javascript
---
ES6 的 `class` 可以看作是一个语法糖，因为它的绝大部分功能，ES5 都可以做到。新的 `class` 写法只是让对象原型的写法更清晰、更像面向对象编程的语法而已。

ES5 的写法：

```javascript
function Point(x, y) {
  this.x = x;
  this.y = y;
}

Point.prototype.toString = function () {
  return '(' + this.x + ', ' + this.y + ')';
}

var p = new Point(1, 2);
```

ES6 的写法：

```javascript
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }

  toString() {
    return '(' + this.x + ', ' + this.y + ')';
  }
}
```

## constructor 方法

`constructor` 方法是默认方法，通过 `new` 命令生成对象实例时，**自动调用该方法**。一个类必须有 `constructor` 方法，**如果没有显式定义，一个空的 `constructor` 方法会被默认添加**。

`constructor` 方法默认返回实例对象（即 `this`），完全可以指定返回另一个对象。但是如果这么做，会导致实例对象不是类的实例，如：

```javascript
class Foo {
  constructor() {
    return Object.create(null);
  }
}

new Foo() instanceof Foo
// false
```

与 ES5 不同，ES6 的类 class 的写法必须用 `new` 调用，否则会报错。而 ES5 的构造函数可以不用 `new` 执行（作为普通函数执行）。

另外，ES6 的 class，实例的属性除非**显式**定义在其本身（即定义在 `this` 对象上），否则都是定义在原型上（即定义在 `class` 上）。所有的实例都共享一个原型对象。

```javascript
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }

  toString() {
    return '(' + this.x + ', ' + this.y + ')';
  }
}

var point = new Point(2, 3);

point.toString();                               // (2, 3)

point.hasOwnProperty('x');                      // true
point.hasOwnProperty('y');                      // true
point.hasOwnProperty('toString');               // false
point.__proto__.hasOwnProperty('toString');     // true

// 所有实例共享一个原型对象
var p1 = new Point(2, 3);
var p2 = new Point(3, 2);

p1.__proto__ === p2.__proto__;                  // true
```

## 取值函数(getter)和存值函数(setter)

在类的内部，可以使用 `get` 和 `set` 关键字，对某个属性设置存值函数和取值函数，拦截该属性的存取行为。

```javascript
class MyClass {
  constructor() {
    // ...
  }
  get prop() {
    return 'getter';
  }
  set prop(value) {
    console.log('setter: ' + value);
  }
}

let inst = new MyClass();

inst.prop = 123;
// 'setter: 123'

inst.prop
// 'getter'
```

上面代码中，`prop` 属性有对应的存值函数和取值函数，因此赋值和读取行为都被自定义了。

存值函数和取值函数是设置在属性的 Descriptor 对象上的。

```javascript
class CustomHTMLElement {
  constructor(element) {
    this.element = element;
  }

  get html() {
    return this.element.innerHTML;
  }

  set html(value) {
    this.element.innerHTML = value;
  }
}

var descriptor = Object.getOwnPropertyDescriptor(
  CustomHTMLElement.prototype, 'html'
);

"get" in descriptor     // true
"set" in descriptor     // true
```

## Class 表达式

与函数一样，类也可以使用表达式的形式定义。

```javascript
const MyClass = class Me {
  getClassName() {
    return Me.name;
  }
}

let inst = new MyClass();
inst.getClassName();    // Me
Me.name;                // ReferenceError: Me is not defined
```

上面的代码定义了一个类，类的名字是 `Me`，但 `Me` 只在 Class 的内部可用，指代当前类，在 Class 外部，这个类只能用 `MyClass` 引用。

如果类的内部没有用到的话，可以省略 `Me`。

采用 Class 表达式，可以写出立即执行的 Class。

```javascript
let person = new class {
  constructor(name) {
    this.name = name;
  }
  sayName() {
    console.log(this.name);
  }
}('张三');

person.sayName();       // 张三
```

## 静态方法

类相当于实例的原型，所有在类中定义的方法，都会被实例继承。如果在一个方法前，加上 `static` 关键字，就表示该方法**不会被实例继承**。

如果静态方法包含 `this` 关键字，这个 `this` 指的是类，而不是实例。

```javascript
class Foo {
  static bar() {
    this.baz();
  }
  static baz() {
    console.log('hello');
  }
  baz() {
    console.log('world');
  }
}

Foo.bar();      // hello
```

上面的代码中，静态方法 `bar` 调用了 `this.baz`，这里的 `this` 指的是 `Foo` 类，而不是 `Foo` 的实例，等同于调用了 `Foo.baz`。另外，这个例子可以看出，**静态方法可以与非静态方法重名**。

## 实例属性的新写法

实例属性除了定义在 `constructor()` 方法里面的 `this` 上面，也可以定义在类的最顶层，这时不需要在实例属性前面加上 `this`。这种新写法的好处是，所有的实例对象的属性都定义在类的头部，看上去比较整齐，一眼就能看出这个类有哪些实例属性。

```javascript
class foo {
  bar = 'hello';
  baz = 'world';

  constructor() {
    // ...
  }
}
```

直接写在顶层就好，不能使用 `var` `let` `const` 这样的关键字，会报错，因为 class 类的写法不是在写一个函数。

而且值得注意的是，写属性与写方法有所不同，**属性是加在实例上的**，也就是 `this` 上的，**而方法是加在原型上的**。

## 类的继承

使用 `extends` 关键字可以实现类的继承，但子类必须在 `constructor` 方法中调用 `super` 方法，否则新建实例时会报错。这是因为子类的自己的 `this` 对象，必须选通过父类的构造函数完成塑造，得到与父类同样的实例属性和方法，然后再对其进行加工，加上子类自己的实例属性和方法。如果不调用 `super` 方法，子类就得不到 `this` 对象。

```javascript
class Point { /* ... */ }

class ColorPoint extends Point {
  constructor() {

  }
}

let cp = new ColorPoint();  // ReferenceError
```

另一个要注意的地方是，在子类的构造函数中，只有调用了 `super` 之后，才可以使用 `this` 关键字，否则会报错。这是因为子类实例的构建，基于父类实例，只有 `super` 方法才能调用父类实例。

```javascript
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }
}

class ColorPoint extends Point {
  constructor(x, y, color) {
    this.color = color;     // ReferenceError
    super(x, y);
    this.color = color;     // 正确
  }
}
```

## super 关键字

`super` 这个关键字，既可以当作函数使用，也可以当作对象使用。

### 作为函数调用

`super` 作为函数调用时，代表**父类**的构造函数，ES6 要求，子类的构造函数必须执行一次 `super` 函数。`super()` 只能用在子类的构造函数之中，用在其他地方就报错。

```javascript
class A {}
class B extends A {
  constructor() {
    super();
  }
}

// 错误的情况
class B extends A {
  m() {
    super();    // 报错
  }
}
```

上面的代码中，子类 `B` 的构造函数之中的 `super()`，虽然代表了父类 `A` 的构造函数，但返回的是子类 `B` 的实例，即 `super` 内的 `this` 指的是 `B` 的实例，因此 `super()` 在这里相当于 `A.prototype.constructor.call(this)`。

### 作为对象使用

`super` 作为对象时，在普通方法中，指向父类的原型对象；在静态方法中，指向父类。

```javascript
class A {
  p() {
    return 2;
  }
}

class B extends A {
  constructor() {
    super();
    console.log(super.p());     // 2
  }
}

let b = new B();
```

上面的代码中，子类 `B` 中的 `super.p()`，就是将 `super` 当作一个对象使用。这时 `super` 在普通方法之中，指向 `A.prototype`，所以 `super.p()` 就相当于 `A.prototype.p()`。

需要注意的是，由于 `super` 指向父类的原型对象，所以定义在父类实例上的方法或属性，是无法通过 `super` 调用的。如果属性定义在父类的原型对象上，`super` 就可以取到。

```javascript
class A {
  constructor() {
    this.p = 2;
  }
}

class B extends A {
  get m() {
    return super.p;
  }
}

let b = new B();
b.m;    // undefined

// 定义在原型上，就能取到
class A {}
A.prototype.x = 2;

class B extends A {
  constructor() {
    super();
    console.log(super.x);   // 2
  }
}

let b = new B();
```

ES6 规定，在子类普通方法中通过 `super` 调用父类的方法时，方法内部的 `this` 指向**当前的子类实例**。

```javascript
class A {
  constructor() {
    this.x = 1;
  }
  print() {
    console.log(this.x);
  }
}

class B extends A {
  constructor() {
    super();
    this.x = 2;
  }
  m() {
    super.print();      // 因为 constructor 中用了 super()，所以此处不报错
  }
}

let b = new B();
b.m();      // 2
```

上面代码中，`super.print()` 虽然调用的是 `A.prototype.print()`，但是 `A.prototype.print()` 内部的 `this` 指向子类 `B` 的实例，导致输出的是 `2`， 而不是 `1`。也就是说，实际执行的是 `super.print.call(this)`。

由于 `this` 指向子类实例，所以如果通过 `super` 对某个属性赋值，这时 `super` 就是 `this`，赋值的属性会变成子类实例的属性。

```javascript
class A {
  constructor() {
    this.x = 1;
  }
}

class B extends A {
  constructor() {
    super();
    this.x = 2;
    super.x = 3;
    console.log(super.x);   // undefined
    console.log(this.x);    // 3
  }
}

let b = new B();
```

上面代码中，`super.x` 赋值为 `3`，这时等同于对 `this.x` 赋值为 `3`。而当读取 `super.x` 的时候，读的是 `A.prototype.x`，所以返回 `undefined`。

如果 `super` 作为对象，用在静态方法之中，这时 `super` 将指向父类，而不是父类的原型对象。

```javascript
class Parent {
  static myMethod(msg) {
    console.log('static', msg);
  }
  myMethod(msg) {
    console.log('instance', msg);
  }
}

class Child extends Parent {
  static myMethod(msg) {
    super.myMethod(msg);
  }
  myMethod(msg) {
    super.myMethod(msg);
  }
}

Child.myMethod(1);      // static 1

var child = new Child();
child.myMethod(2);      // instance 2
```

上面代码中，`super` 在静态方法之中指向父类，在普通方法之中指向父类的原型对象。

另外，在子类的静态方法中通过 `super` 调用父类的方法时，方法内部的 `this` 指向当前的子类，而不是子类的实例。

```javascript
class A {
  constructor() {
    this.x = 1;
  }
  static print() {
    console.log(this.x);
  }
}

class B extends A {
  constructor() {
    super();
    this.x = 2;
  }
  static m() {
    super.print();
  }
}

B.m();      // undefined
B.x = 3;
B.m();      // 3
```

上面代码中，静态方法 `B.m` 里面，`super.print` 指向父类的静态方法 `print`。**这个方法里面的 `this` 指向的是 `B`，而不是 `B` 的实例**。一开始调用 `B.m`，因为 `B` 没有这个属性（静态属性），所以返回 `undefined`；但是当给 `B.x` 赋值为 `3` 的话，那么再调用 `B.m` 返回 `3`。

注意，使用 `super` 的时候，必须显式指定是作为函数、还是作为对象使用，否则会报错。

```javascript
class A {}

class B extends A {
  constructor() {
    super();
    console.log(super);     // 报错
  }
}
```

上面代码中，`console.log(super)` 当中的 `super`，无法看出是作为函数使用，还是作为对象使用，所以 JS 引擎解析代码时会报错。但如果能清晰表明 `super` 的数据类型，就不会报错。

```javascript
class A {}

class B extends A {
  constructor() {
    super();
    console.log(super.valueOf() instanceof B);      // true
  }
}

let b = new B();
```

上面代码中，`super.valueOf()` 表明 `super` 是一个对象，因此就不会报错。同时，由于 `super` 使得 `this` 指向 `B` 的实例，所以 `super.valueOf()` 返回的是一个 `B` 的实例。

最后，由于对象总是继承其他对象的，所以可以在任意一个对象中，使用 `super` 关键字。

```javascript
var obj = {
  toString() {
    return 'MyObject: ' + super.toString();
  }
};

obj.toString();     // MyObject: [object object]
```
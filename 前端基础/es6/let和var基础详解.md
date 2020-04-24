---
title: let和var基础详解
data: 2020-04-23 20:00
categories: 
- js

tags:
- es6
- js
- let

toc: true

---

### let是什么
> ES6新增了`let`命令，用来声明变量。它的用法类似于`var`，但是所声明的变量，只在`let`命令所在的代码块内有效。
```javascript
{
  let a = 10;
  var b = 1;
}
console.log(a) // ReferenceError: a is not defined.
console.log(b) // 1
```
上面代码在代码块之中，分别用`let`和`var`声明了两个变量。然后在代码块之外调用这两个变量，结果`let`声明的变量报错，`var`声明的变量返回了正确的值。这表明，`let`声明的变量只在它所在的代码块有效。


`for`循环的计数器，就很合适使用let命令。
```javascript
for (let i = 0; i < 10; i++) {}

console.log(i);
//ReferenceError: i is not defined
```
上面代码中，计数器i只在`for`循环体内有效，在循环体外引用就会报错。

下面的代码如果使用`var`，最后输出的是10。
```javascript
var a = [];
for (var i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[6](); // 10
```
上面代码中，变量i是`var`声明的，在全局范围内都有效。所以每一次循环，新的i值都会覆盖旧值，导致最后输出的是最后一轮的i的值。

如果使用`let`，声明的变量仅在块级作用域内有效，最后输出的是6。
```javascript
var a = [];
for (let i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[6](); // 6
```

上面代码中，变量i是`let`声明的，当前的i只在本轮循环有效，所以每一次循环的i其实都是一个新的变量，所以最后输出的是6。
 

### 为什么要使用它
####  不存在变量提升
`let`不像`var`那样会发生“变量提升”现象。所以，变量一定要在声明后使用，否则报错。
```javascript
console.log(foo); // 输出undefined
console.log(bar); // 报错ReferenceError

var foo = 2;
let bar = 2;
```

上面代码中，变量`foo`用`var`命令声明，会发生变量提升，即脚本开始运行时，变量`foo`已经存在了，但是没有值，所以会输出`undefined`。变量`bar`用`let`命令声明，不会发生变量提升。这表示在声明它之前，变量`bar`是不存在的，这时如果用到它，就会抛出一个错误。

#### 暂时性死区
只要块级作用域内存在`let`命令，它所声明的变量就“绑定”（binding）这个区域，
不再受外部的影响。
```javascript
var tmp = 123;
if (true) {
  tmp = 'abc'; // ReferenceError
  let tmp;
}
```

上面代码中，存在全局变量`tmp`，但是块级作用域内`let`又声明了一个局部变量`tmp`，
导致后者绑定这个块级作用域，所以在`let`声明变量前，对`tmp`赋值会报错。

ES6明确规定，如果区块中存在`let`和`const`命令，这个区块对这些命令声明的变量，
从一开始就形成了封闭作用域。凡是在声明之前就使用这些变量，就会报错。

总之，在代码块内，使用`let`命令声明变量之前，该变量都是不可用的。
这在语法上，称为“暂时性死区”（temporal dead zone，简称TDZ）。
```javascript
if (true) {
  // TDZ开始
  tmp = 'abc'; // ReferenceError
  console.log(tmp); // ReferenceError

  let tmp; // TDZ结束
  console.log(tmp); // undefined

  tmp = 123;
  console.log(tmp); // 123
}

```
上面代码中，在`let`命令声明变量tmp之前，都属于变量`tmp`的“死区”。
“暂时性死区”也意味着`typeof`不再是一个百分之百安全的操作。
```javascript
typeof x; // ReferenceError
let x;
```

上面代码中，变量x使用`let`命令声明，所以在声明之前，都属于x的“死区”，
只要用到该变量就会报错。因此，`typeof`运行时就会抛出一个`ReferenceError`。
作为比较，如果一个变量根本没有被声明，使用typeof反而不会报错。
```javascript
typeof undeclared_variable // "undefined"
```

上面代码中，`undeclared_variable`是一个不存在的变量名，
结果返回“undefined”。所以，在没有`let`之前，`typeof`运算符是百分之百安全的，
永远不会报错。现在这一点不成立了。这样的设计是为了让大家养成良好的编程习惯，
变量一定要在声明之后使用，否则就报错。

```javascript
function bar(x = y, y = 2) {
  return [x, y];
}

bar(); // 报错
```
上面代码中，调用`bar`函数之所以报错（某些实现可能不报错），
是因为参数x默认值等于另一个参数y，而此时y还没有声明，属于”死区“。
如果y的默认值是x，就不会报错，因为此时x已经声明了。

有些“死区”比较隐蔽，不太容易发现。
ES6规定暂时性死区和`let`、`const`语句不出现变量提升，主要是为了减少运行时错误，
防止在变量声明前就使用这个变量，从而导致意料之外的行为。这样的错误在ES5是很常见的，
现在有了这种规定，避免此类错误就很容易了。

总之，暂时性死区的本质就是，只要一进入当前作用域，所要使用的变量就已经存在了，
但是不可获取，只有等到声明变量的那一行代码出现，才可以获取和使用该变量。

#### 不允许重复声明
let不允许在相同作用域内，重复声明同一个变量。

```javascript
// 报错
function func() {
  let a = 10;
  var a = 1;
}

// 报错
function func() {
  let a = 10;
  let a = 1;
}
```

因此，不能在函数内部重新声明参数。

```javascript
function func(arg) {
  let arg; // 报错
}

function func(arg) {
  {
      console.log(arg);
    let arg; // 不报错
  }
}
```

#### 块级作用域
##### 为什么需要块级作用域？
ES5只有全局作用域和函数作用域，没有块级作用域，这带来很多不合理的场景。

第一种场景，内层变量可能会覆盖外层变量。

```javascript
var tmp = new Date();

function f() {
  console.log(tmp); // 变量提升函数先于全局
  if (false) {
    var tmp = "hello world";
  }
}

f(); // undefined
```
上面代码中，函数f执行后，输出结果为`undefined`，原因在于变量提升，
导致内层的tmp变量覆盖了外层的tmp变量。

第二种场景，用来计数的循环变量泄露为全局变量。

```javascript
var s = 'hello';

for (var i = 0; i < s.length; i++) {
  console.log(s[i]); // 真的可以做数组来取
}

console.log(i); // 5
```
上面代码中，变量i只用来控制循环，但是循环结束后，它并没有消失，泄露成了全局变量。

##### ES6的块级作用域
let实际上为JavaScript新增了块级作用域。
```javascript
function f1() {
  let n = 5;
  if (true) {
    let n = 10;
  }
  console.log(n); // 5
}
```

上面的函数有两个代码块，都声明了变量n，运行后输出5。这表示外层代码块不受内层代码块的影响。
如果使用`var`定义变量n，最后输出的值就是10。

ES6允许块级作用域的任意嵌套。
```javascript
{
    {
        {let insane = 'Hello World'}
    }
};

```
上面代码使用了一个三层的块级作用域。外层作用域无法读取内层作用域的变量。

```javascript
{{{{
  {let insane = 'Hello World'}
  console.log(insane); // 报错
}}}};
```


内层作用域可以定义外层作用域的同名变量。
```javascript
{{{{
  let insane = 'Hello World';
  {let insane = 'Hello World'}
}}}};
```

块级作用域的出现，实际上使得获得广泛应用的立即执行函数表达式（IIFE）不再必要了。

```javascript
// IIFE 写法 匿名函数自调用(IIFE) 变量只在函数里有用
(function () {
  var tmp = ...;
  ...
}());

// 块级作用域写法
{
  let tmp = ...;
  ...
}
```

##### 块级作用域与函数声明

函数能不能在块级作用域之中声明，是一个相当令人混淆的问题。

ES5规定，函数只能在顶层作用域和函数作用域之中声明，不能在块级作用域声明。
```javascript
// 情况一
if (true) {
  function f() {}
}

// 情况二
try {
  function f() {}
} catch(e) {
}
```

上面代码的两种函数声明，根据ES5的规定都是非法的。

但是，浏览器没有遵守这个规定，为了兼容以前的旧代码，还是支持在块级作用域之中声明函数，
因此上面两种情况实际都能运行，不会报错。不过，“严格模式”下还是会报错。
```javascript
// ES5严格模式
'use strict';
if (true) {
  function f() {}
}
// 报错
```
ES6 引入了块级作用域，明确允许在块级作用域之中声明函数。
```javascript
// ES6严格模式
'use strict';
if (true) {
  function f() {}
}
// 不报错
```
ES6 规定，块级作用域之中，函数声明语句的行为类似于`let`，在块级作用域之外不可引用。
```javascript
function f() { console.log('I am outside!'); }
(function () {
  if (false) {
    // 重复声明一次函数f
    function f() { console.log('I am inside!'); }
  }
  
  f();
}());
```

上面代码在 ES5 中运行，会得到“I am inside!”，因为在if内声明的函数f会被提升到函数头部，实际运行的代码如下。
```javascript
// ES5版本
function f() { console.log('I am outside!'); }
(function () {
  function f() { console.log('I am inside!'); }
  if (false) {
  }
  f();
}());
```

ES6 的运行结果就完全不一样了，会得到“I am outside!”。因为块级作用域内声明的函数类似于`let`，
对作用域之外没有影响，实际运行的代码如下。
```javascript
// ES6版本
function f() { console.log('I am outside!'); }
(function () {
  f();
}());
```

很显然，这种行为差异会对老代码产生很大影响。为了减轻因此产生的不兼容问题，ES6在附录B里面规定，
浏览器的实现可以不遵守上面的规定，有自己的行为方式。

允许在块级作用域内声明函数。
函数声明类似于`var`，即会提升到全局作用域或函数作用域的头部。
同时，函数声明还会提升到所在的块级作用域的头部。
注意，上面三条规则只对ES6的浏览器实现有效，其他环境的实现不用遵守，
还是将块级作用域的函数声明当作let处理。

前面那段代码，在 Chrome 环境下运行会报错。
```javascript
// ES6的浏览器环境
function f() { console.log('I am outside!'); }
(function () {
  if (false) {
    // 重复声明一次函数f
    function f() { console.log('I am inside!'); }
  }

  f();
}());
// Uncaught TypeError: f is not a function
```

上面的代码报错，是因为实际运行的是下面的代码。
```javascript

```
// ES6的浏览器环境
function f() { console.log('I am outside!'); }
(function () {
  var f = undefined;
  if (false) {
    function f() { console.log('I am inside!'); }
  }

  f();
}());
// Uncaught TypeError: f is not a function
考虑到环境导致的行为差异太大，应该避免在块级作用域内声明函数。如果确实需要，
也应该写成函数表达式，而不是函数声明语句。
```javascript
// 函数声明语句
{
  let a = 'secret';
  function f() {
    return a;
  }
}

// 函数表达式
{
  let a = 'secret';
  let f = function () {
    return a;
  };
}
```

另外，还有一个需要注意的地方。ES6的块级作用域允许声明函数的规则，
只在使用大括号的情况下成立，如果没有使用大括号，就会报错。
```javascript
// 不报错
'use strict';
if (true) {
  function f() {}
}

// 报错
'use strict';
if (true)
  function f() {}
```

#### do 表达式
本质上，块级作用域是一个语句，将多个操作封装在一起，没有返回值。
```javascript
{
  let t = f();
  t = t * t + 1;
}
```

上面代码中，块级作用域将两个语句封装在一起。但是，在块级作用域以外，没有办法得到t的值，
因为块级作用域不返回值，除非t是全局变量。

现在有一个提案，使得块级作用域可以变为表达式，也就是说可以返回值，
办法就是在块级作用域之前加上do，使它变为do表达式。
```javascript
let x = do {
  let t = f();
  t * t + 1;
};
```

上面代码中，变量x会得到整个块级作用域的返回值。

### 我的总结
1.变量提升
- `let`没有变量提升，`var`有变量提升 
- 提升优先级： 变量 \> 函数 \> 参数 \> 提升
 
2.暂时性死区
- 在`let`已声明变量的作用域里变量已存在，但不能使用，在声明语句后面才能正常使用
- es6用来规范变量的使用

3.不允许重复声明
- `let`变量在同一作用域只能声明一次

4.块级作用域
- `let`只能在当前作用域使用


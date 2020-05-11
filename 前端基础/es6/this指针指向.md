---

title: 彻底理解this指向
date: 2020-05-06 21:22:59

categories:
- es6

tags:
- this
- es6

toc: true

---

**this**：当前执行代码的环境对象，在非严格模式下，总是指向一个对象，在严格模式下可以是任意值，
[具体官方详细介绍](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/this)

**作用**：便捷的获取使用所指向环境的“属性”，“方法”

<!--more-->

----


### 全局`window`
在全局作用域下使用`this`,得到的是顶层对象`window`
```javascript
this
// Window {parent: Window, postMessage: ƒ, blur: ƒ, focus: ƒ, close: ƒ, …}
console.log(this === window); // true
```

然而，在严格模式下，`this`将保持他进入执行环境时的值，所以下面的`this`将会默认为`undefined`。
```javascript
function f2(){
  "use strict"; // 这里是严格模式
  return this;
}

f2() === undefined; // true
```

> 所以，在严格模式下，如果 `this` 没有被执行环境（execution context）定义，那它将保持为 `undefined`。

[请看第5点](#这里是第五点)
### 函数

#### 普通函数
普通函数这里是指使用`function` 声明的函数
```javascript
var obj ={
	func:function(){
		console.log(this)
	}
}
obj.func()
// VM7926:3 {func: ƒ}
```
> 上面代码可以看出普通函数`this`指向当前调用的对象obj. <u>`this`对象是在运行时基于函数的执行环境绑定的：
在全局函数中，`this`指向的是`window`；当函数被作为某个对象的方法调用时，`this`就等于那个对象</u>。

#### 闭包函数
`this`指向`retrun`出来函数的执行环境
```javascript
var name = "window";
var object = {
    name : "object",
    getNameFunc : function(){
        return function(){
            return this.name;
        };
    }
};
var obj = {
    name : "obj",
    getNameFunc : object.getNameFunc()
};
function foo() {
        return this.name;
}
console.log(foo());//Window
console.log(object.getNameFunc()());  //Window
console.log(obj.getNameFunc());//obj

```

> 闭包函数无法直接访问包含他的函数的`this`对象, 因为二者的`this`指向是不一样的,
外部函数的`this`指向调用他的对象, 闭包内部函数的`this`指向了全局对象, 其实并不难理解.
实际上, <u>当调用object.getNameFunc()时, 就像全局对象返回了一个函数, 返回的这个函数和foo函数其实并无两样</u>.
在全局中调用这个返回的函数时, 函数的`this`自然就指向了全局对象, 就好像调用foo函数一样.
 <u>而当object.getNameFunc()在obj内部时, 返回的闭包函数就成了obj 的属性, 此时闭包函数的this就指向了obj对象</u>.

#### 箭头函数
箭头函数中`this`指向当前封闭环境，在全局代码中，它将被设置为全局对象
```javascript
var object = {
    name: "object",
    getNameFunc: function() {      
        return ()=>{
            return this.name;
        };
    }
};
var obj = {
    name : "obj",
    getNameFunc : object.getNameFunc()
};
console.log(object.getNameFunc()());  //object
console.log(obj.getNameFunc());//object
```
> 上面代码看出，**this**一直指向的是**object**这个封闭的词法环境

#### 构造函数
这个就不做太多解释了，当一个函数用作构造函数时（使用`new`关键字），它的`this`被绑定到正在构造的新对象，
不了解看[官方介绍](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/this#%E4%BD%9C%E4%B8%BA%E6%9E%84%E9%80%A0%E5%87%BD%E6%95%B0)

### 原型链中的 this
对于在对象原型链上某处定义的方法，同样的概念也适用。
如果该方法存在于一个对象的原型链上，那么`this`指向的是调用这个方法的对象，就像该方法在对象上一样。
```javascript
var o = {
  f: function() { 
    return this.a + this.b; 
  }
};
var p = Object.create(o);
p.a = 1;
p.b = 4;

console.log(p.f()); // 5
```


### 更换绑定环境
把 `this` 的值从一个环境传到另一个环境

#### call 和 apply
```javascript
var obj = {a: 'Custom'};
// 这个属性是在global对象定义的。
var a = 'Global';

function whatsThis(arg) {
  return this.a;  // this的值取决于函数的调用方式
}
whatsThis();          // 'Global'
whatsThis.call(obj, 'is_a');  // 'Custom'
whatsThis.apply(obj, ['is_a']);  // 'Custom'
```
> `apply` 和 `call` 的差别只是参数传递方式不一样

#### bind
es5就加入了改方法，同样是更改`this`指向环境，不过`bind` 是调用f.bind(someObject)会创建一个与f具有相同函数体和作用域的函数，
但是在这个新函数中，<u>`this`将永久地被绑定到了`bind`的第一个参数，无论这个函数是如何被调用的</u>
```javascript
function f(){
  return this.a;
}

var g = f.bind({a:"azerty"});
console.log(g()); // azerty

var h = g.bind({a:'yoo'}); // bind只生效一次！
console.log(h()); // azerty

var o = {a:37, f:f, g:g, h:h};
console.log(o.a, o.f(), o.g(), o.h()); // 37, 37, azerty, azerty
```

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

**this**：this是一个特殊的对象，在不同的情况之下，指向不同的位置，可以看做`this`是指向当前执行环境（函数，对象），
**作用**：便捷的获取使用所指向环境的“属性”，“方法”

<!--more-->

----


### 全局`window`
在全局作用域下使用`this`,得到的是顶层对象`window`
```javascript
this
// Window {parent: Window, postMessage: ƒ, blur: ƒ, focus: ƒ, close: ƒ, …}
```

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
> 上面代码可以看出普通函数`this`指向当前调用的对象obj

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
```javascript
var object = {
    name : "object",
    getNameFunc : ()=>{
        return function(){
            return this.name;
        };
    }
};
var obj = {
    name : "obj",
    getNameFunc : object.getNameFunc()
};
console.log(object.getNameFunc()());  //Window
console.log(obj.getNameFunc());//obj
```
> 箭头函数`this`则类似于就近原则，如果有嵌套则绑定到最近的一层对象上，如果没有嵌套，谁调用指向谁

#### 3
这里是第2点
这里是第2点
这里是第2点

这里是第2点

#### 4
这里是第2点
这里是第2点
这里是第2点

这里是第2点

#### 这里是第五点


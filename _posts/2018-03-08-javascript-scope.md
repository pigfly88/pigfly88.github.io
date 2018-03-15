---
layout: post
title:  JavaScript一看就懂(1)作用域
categories: javascript
---

### 函数级作用域

**1.函数外声明的变量为全局变量，函数内可以直接访问全局变量**：

```javascript
var global_var = 10; //全局变量
function a(){
    alert(global_var); //全局变量在函数内可访问
}
a(); //10
```

**2.JavaScript变量的作用域是*函数级*的，只有函数可以产生新的作用域，而非块级**：

```javascript
function a(){ //函数
    if(true){ //块
        var x = 1;
    }
    if(false){
        var y = 2;
    }
    alert(x); //1
    alert(y); //undefined
}
a();
```
变量x虽然在块语句(if)中声明并赋值，但它的作用域是函数a，所以在函数a的任何位置它都可访问。

有意思的是y变量的声明和赋值虽然在false块语句里，却仍然打印出undefined而不是报错，因为JavaScript会把所有变量声明提前到函数开头，称为*变量提升*：

```javascript
function a(){
    alert(x);
    var x = 1;
    alert(x);
}
a(); //undefined 1

//以上代码经过JavaScript变量提升后实际上是这个样子的：
function a(){
    var x;
    alert(x);
    x = 1;
    alert(x);
}
```

需要注意的是，在函数内声明变量一定要使用var，否则是声明了一个全局变量：

```javascript
function test(){
    a = 1;
}
test();
alert(a); //1
```

**3.嵌套函数可以访问外围函数的变量**

```javascript
function a(){
    var x = 1;
    
    function b(){
        alert(x);
        var y = 2;
        
        function c(){
            alert(x);
            alert(y);
        }
        c();
    }
    b();
}
a(); // 1 1 2
```

需要注意的是，嵌套函数里如果有同名变量，那么访问到的是嵌套函数里的变量
```javascript
function a(){
    var x = 1;
    
    function b(){
        var x = 2;
        alert(x);
    }
    b();
    alert(x);
}
a(); //2 1
```

### 如何做到块级作用域

通过上面的例子我们了解到JavaScript变量作用域是函数级的，但有时候我们想用临时变量怎么办呢？

通过*IIFE(立即执行函数表达式)*来实现：

```javascript
function a(){
    if(true){
        (function(){ //IIFE开始
            var x = 1;
            alert(x); //1
        }()); //IIFE结束
        //alert(x); //这儿访问不到
    }
    //alert(x); //这儿访问不到
}
a();
```
这样做的好处是不会造成变量污染，用完就没了，大家商量好，过了今晚就不再联系。

### 作用域链

每当JavaScript解释器进入一个函数，它都会看一下附近有哪些局部变量，把它们保存到该函数的*variables*对象中，并创建一个*scope*属性来指向外部的variables对象
```javascript
1. var x=1;
2. function a() {
3.    var y = 2;
4.    function b() {
5.        var z = 3;
6.        alert(x+y+z);
7.    }
8.    b();
9. }
10.a();
```

1.JavaScript把所有全局对象(变量x和函数a)放到global variables对象中

global variables: x, a()

2.发现a是个函数，它需要一个scope属性来指向外部的variables(全局)，同时把变量保存起来

a.scope -> global variables

a.variables: y, b()

3.来到第4行，发现b是个函数，它需要一个scope属性来指向它所在的作用域(a)

b.scope -> a.variables

b.variables: z

![js-scope](https://pigfly88.github.io/images/js-scope.png?v=2)

### 参考资料
> 1.   JavaScript for PHP Developers
> 2.   深入理解JavaScript
> 3.   [You Don't Know JS](https://github.com/getify/You-Dont-Know-JS/tree/1ed-zh-CN/scope%20%26%20closures)



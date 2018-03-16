---
layout: post
title:  JavaScript一看就懂(4)this
categories: javascript
---

### what is this?

this取决于调用方式，它不是指向自己，跟作用域也没半毛钱关系。

### 函数级作用域

#### 全局环境调用

```javascript
var i = 1;
this.j = 2;
console.log(this, this.i, this.j); //window 1 2
```
this默认为全局对象，在网页上是window对象

#### 简单函数调用

```javascript
var x = 1;
foo();
function foo(){
    console.log(this, this.x); //window 1
}
```
直接调用函数的方式，this是全局对象

#### 对象方法调用

```javascript
var x = 1;
var obj = {
    x: 2,
    foo: function(){
        console.log(this, this.x); //"obj" 2
    }
};
obj.foo();
```
这时，this就是这个方法所属的对象

#### 构造函数调用
```javascript
function Foo(x){
    this.x = x;
}
var f1 = new Foo(1);
var f2 = new Foo(2);
console.log(f1.x, f2.x); //1 2
```
这时，this就是这个对象的实例

### 参考资料
> 1.   深入理解JavaScript
> 2.   [深入理解JavaScript系列（13）：This? Yes,this!](http://www.cnblogs.com/TomXu/archive/2012/01/17/2310479.html)



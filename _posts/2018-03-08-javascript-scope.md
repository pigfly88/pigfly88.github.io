---
layout: post
title:  JavaScript作用域
categories: javascript
---

### 全局or局部

- 函数外声明的变量为全局变量，函数内声明的变量为局部变量
- 函数内可以直接使用全局变量
- 块语句（如：if、for、while）里面声明的变量并非局部变量

```javascript
var global_var = 1; //全局
if(false){
	var false_var = 2; //还是全局
}

if(true){
	var true_var = 3; //还是全局
}

function test(){
	var local_var = 4; //局部
	return local_var+global_var;
}

console.log(false_var, true_var); //undefined 1
test(); //5
console.log(local_var); //local_var is not defined
```

我们可以看到false_var是undefined，代表它已经声明，不过因为条件是false，没有执行赋值

需要注意的是，在函数内声明变量一定要使用var，否则是声明了一个全局变量
```javascript
function test(){
	a = 1;
	return a;
}
test(); //1
console.log(a); //1
```

### 作用域链

变量是以函数为作用域，只有函数可以产生新的作用域。

当一个函数里面还有另外一个函数的时候，就涉及到查找变量的问题
```javascript
var x=1;
function outer() {
	var y = 2;
	function inner() {
		var z = 3;
		alert(x+y+z);
	}
	inner();
}
outer();
```

inner函数出现了x、y两个变量，JavaScript由内而外进行搜索，首先看看inner函数里面有没有定义，没有则去查找outer函数，再没有找到就只能去最外层查找全局变量了。


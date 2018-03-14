---
layout: post
title:  JavaScript一看就懂(2)闭包
categories: javascript
---

认识闭包之前需要先了解作用域，如果你对作用域还没有足够了解，请移步[JavaScript一看就懂(1)作用域](https://pigfly88.github.io/javascript/2018/03/08/javascript-scope.html)

### 什么是闭包

我们可以先简单认为：**一个函数a定义在另一个函数b里面，这个函数a就是闭包**：
```javascript
function b() {
    ...
    function a() { //闭包
        ...
    }
    ...
}
```

**另外，函数a能够直接读取函数b的变量x**：
```javascript
function b() {
    var x = 1;
    function a() {
        alert(x);
    }
    a();
}
b(); //1
```
这其实不是什么新鲜事，这是由作用域决定的，可以参考[JavaScript一看就懂(1)作用域](https://pigfly88.github.io/javascript/2018/03/08/javascript-scope.html)

上面的例子只是为了简单地认识闭包的组成部分，但却不是真正意义上的闭包，下面才是正宗长沙臭豆腐，哦不，**正宗闭包**：
```javascript
function b() {
    var x = 1;
    function a() {
        alert(x++);
    }
    return a;
}
var func = b();
func(); //1
func(); //2
func(); //3
```
b()这次没有直接调用a()，而是返回了函数a给变量func，而func()才是真正调用a()的地方，等于是跳出了原来的圈子在外围被调用了，而函数a竟然能神奇地记住变量x，这就是闭包的魔力。

所以什么是闭包？结合上面的代码来解释的话：

> 闭包就是无论函数a在哪里执行，它都能联系上变量x

纯字面上来理解的话：

> 一个函数没有在它定义时所在的作用域里执行，但它仍能访问那个作用域里的变量

也就是书上说的：
> 函数以及它所连接的周围作用域中的变量即为闭包

> 闭包：使得函数可以维持其创建时所在的作用域。如果一个函数离开了它被创建时的作用域，它还是会与这个作用域以及其外部的作用域的变量相关联

> 闭包就是函数能够记住并访问它的词法作用域，即使当这个函数在它的词法作用域之外执行时

> 闭包就是当一个函数即使是在它的词法作用域之外被调用时，也可以记住并访问它的词法作用域

### 为什么要用闭包

1.回调

我们其实经常用到闭包，可能自己都不知道，比如下面这个点击按钮获取验证码的例子：
```javascript
function countDown(){
    ...
    document.getElementById('getCode').onclick = function(){ //闭包
        ...
    };
}
```
页面加载完成时，获取验证码的按钮被绑定了一个点击回调事件（没有执行），当用户点击时，才真正被执行

完整代码：
```html
<!DOCTYPE html>
<html lang="zh-cn">
<head></head>
<body>
<button type="button" id="getCode">获取验证码</button>

<script>
countDown(9);

function countDown(time_wait){
    time_wait = time_wait || 60; //等待时间
    var time_left = time_wait; //剩余等待时间
    document.getElementById('getCode').onclick = function(){ //闭包
        var btn = this;
        btn.disabled = true;
        btn.innerHTML=time_left--;
        var count = setInterval(function(){ //闭包
            if(time_left<1){
                btn.disabled = false;
                btn.innerHTML="获取验证码";
                time_left = time_wait;
                clearInterval(count);
            }else{
                btn.innerHTML=time_left--;
            }
        },1000);
    };
}
</script>
</body>
</html>
```
当用户点击按钮时，执行了这个回调函数并且它能够读取到外围的time_wait和time_left变量。

细心的朋友可能已经发现这里出现两个闭包了，一个是onclick，还有一个是setInterval定时函数。

2.面向对象

当然还有一个更重要的原因需要它，那就是*OO*，把上面发送验证码的代码改造一下变成面向对象风格
```javascript
var c1 = CountDown(document.getElementById('getCode'), 5);
c1.run();

function CountDown(btn, time_wait) {
    time_wait = time_wait || 60;
    var time_left = time_wait; //剩余等待时间
    
    function resetBtn(){
        btn.disabled = false;
        btn.innerHTML="获取验证码";
    }

    function run(){
        btn.onclick = function(){
            btn.innerHTML=time_left--;
            btn.disabled = true;
            var count = setInterval(function(){
                
                if(time_left<1){
                    resetBtn();
                    time_left = time_wait;
                    clearInterval(count);
                }else{
                    btn.innerHTML=time_left--;
                }
            },1000);
        };
    }
    
    //相当于公共方法
    return {
        run: run
    };
}
```
只有在return返回的对象里面的run方法才能对外服务，而resetBtn()相当于是私有方法

所以回过头来看问题，为什么要用闭包？
- **回调（setTimeout、setInterval、onclick...）**
- **面向对象**

### 闭包会带来什么问题？

**1.循环**

```javascript
<!DOCTYPE html>
<html lang="zh-cn">
<head></head>
<body>
    <input type="button" id="btn1" value="1" />
    <input type="button" id="btn2" value="2" />
    <input type="button" id="btn3" value="3" />

    <script>
    window.addEventListener("load", function() {
        for (var i = 1; i < 4; i++) {
            var btn = document.getElementById("btn" + i);

            btn.onclick = function() {
                alert(i);
            };
        }
    });
    </script>
</body>
</html>
```
乍一看感觉就是对应地弹出1,2,3，但现实是无论你点击哪个按钮，都会弹出4，因为onclick绑定的函数是个闭包，页面加载后for循环开始执行，当你点击按钮的时候，for循环已经执行完了，此时i值为4，而闭包拿到的i值当前的i值，而不是循环中的i值，所以弹出的都是4。

要解决这个问题，可以把打印的代码提炼到另一个函数里面，这样就会产生一个新的作用域，从而避开闭包带来的共享：
```javascript
function show(i){
    return function(){
        alert(i);
    }
}

window.addEventListener("load", function() {
    for (var i = 1; i < 4; i++) {
        var btn = document.getElementById("btn" + i);

        btn.onclick = show(i);
    }
});
```

或者用立即执行函数表达式(IIFE)：
```javascript
window.addEventListener("load", function() {
    for (var i = 1; i < 4; i++) {
        var btn = document.getElementById("btn" + i);

        btn.onclick = (function(i) {
            return function(){
                alert(i);
            }
        }(i));
    }
});
```

当然还有一种更牛逼的方式：
```javascript
window.addEventListener("load", function() {
    for (let i = 1; i < 4; i++) {
        var btn = document.getElementById("btn" + i);

        btn.onclick = function() {
            alert(i);  
        };
    }
});
```

**2.构造器**
滥用闭包可能会导致脚本执行缓慢并消耗不必要的内存。

方法应该定义到对象的原型，而不要定义到对象的构造器中。原因是这将导致每次对象实例化时，方法都会被重新定义一次。
```javascript
function User(name) {
    this.name = name;

    this.getName = function() {
        return this.name;
    };
}
```
改成如下，继承的原型可以为所有实例共享：
```javascript
function User(name) {
    this.name = name;
}

User.prototype.getName = function() {
    return this.name;
};
```


### 参考资料
> 1.   [阮一峰>学习Javascript闭包](http://www.ruanyifeng.com/blog/2009/08/learning_javascript_closures.html)
> 2.   [MDN>Web技术文档>JavaScript>闭包](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Closures)
> 3.   [JavaScript Closures Demystified](https://www.sitepoint.com/javascript-closures-demystified/)
> 4.   JavaScript for PHP Developers
> 5.   深入理解JavaScript
> 6.   [secrets of javascript closures](https://kryogenix.org/code/browser/secrets-of-javascript-closures/secrets_of_javascript_closures.pdf)
> 7.   [You Don't Know JS](https://github.com/getify/You-Dont-Know-JS/tree/1ed-zh-CN/scope%20%26%20closures)
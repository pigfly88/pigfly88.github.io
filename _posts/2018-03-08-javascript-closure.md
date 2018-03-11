---
layout: post
title:  JavaScript闭包
categories: javascript
---

### 什么是闭包
**一个函数a定义在另一个函数b里面，这个函数a就是闭包**

```javascript
function b() {
    ...
    function a() { //闭包
        ...
    }
    ...
}
```

**同时，函数a能够直接读取函数b的变量，这个就是闭包的特性**
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

再来看个更有意思的例子：
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
函数b这次不直接执行a，而是返回了a，所以func是函数a实例的引用，而func()竟然能神奇地记住x，这得益于JavaScript的闭包机制。

综上所述，结合以上代码来看，**闭包就是函数a和变量x**，也就是书上说的：
> 函数以及它所连接的周围作用域中的变量即为闭包

> 闭包：使得函数可以维持其创建时所在的作用域。如果一个函数离开了它被创建时的作用域，它还是会与这个作用域以及其外部的作用域的变量相关联。

### 为什么要用闭包

我们其实经常用到闭包，可能自己都不知道，比如下面这个例子，一个初始化函数里面包含一个按钮点击事件：
```javascript
function init(){
    ...
    document.getElementById('btn').onclick = function(){ //闭包
        ...
    };
}
```

一个详细点的例子，比如发送验证码
```html
<!DOCTYPE html>
<html lang="zh-cn">
<head></head>
<body>
<button type="button" id="getCode">获取验证码</button>

<script>
init();

function init(){
    var time_wait = 60; //等待时间
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
可以看到在onclick函数里面是可以正常操作time_wait和time_left变量的，细心的朋友可能已经发现这里出现两个闭包了，一个是onclick，还有一个是setInterval定时函数。

所以回过头来看问题，为什么要用闭包？
- **JavaScript语言特性需要（setTimeout、setInterval...）**
- **事件绑定需要（onclick...）**

当然还有一个更重要的原因需要它，那就是**OO**，把上面发送验证码的代码改造一下变成面向对象风格
```javascript
var c = new CountDown(document.getElementById('getCode'), 5);
c.run();

function CountDown(btn, time_wait) {
    var time_left = time_wait; //剩余等待时间
    
    //private
    function resetBtn(){
        btn.disabled = false;
        btn.innerHTML="获取验证码";
    }
    
    //public
    this.run = function(){
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
    };
}
```

### 什么时候不要用闭包
滥用闭包可能会导致脚本执行缓慢并消耗不必要的内存

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

            btn.addEventListener("click", function() {
                alert(i);
            });
        }
    });
    </script>
</body>
</html>
```
无论你点击哪个按钮，都会弹出4，因为click函数是个闭包，页面加载后循环开始执行，当你点击按钮的时候，循环已经执行完了，此时i值为4，而闭包记得它周围的变量，所以弹出的都是4。
要解决这个问题，可以把打印的代码提炼到另一个函数里面，这样就会产生一个新的作用域：
```javascript
function show(i){
    return function(){
        alert(i);
    }
}

window.addEventListener("load", function() {
    for (var i = 1; i < 4; i++) {
        var btn = document.getElementById("btn" + i);

        btn.addEventListener("click", show(i));
    }
});
```
或者用立即执行函数表达式：
```javascript
window.addEventListener("load", function() {
    for (var i = 1; i < 4; i++) {
        var btn = document.getElementById("btn" + i);

        btn.addEventListener("click", function(i) {
            return function(){
                alert(i);
            }
        }(i));
    }
});
```

**2.构造器**

方法应该关联于对象的原型，而不要定义到对象的构造器中。原因是这将导致每次对象实例化时，方法都会被重新定义一次。
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
---
layout: post
title:  JavaScript闭包
categories: javascript
---

### 什么是闭包
一个函数a定义在另一个函数b里面，这个函数a就是闭包

```javascript
function b() {
    ...
    function a() { //闭包
        ...
    }
    ...
}
```

同时，函数a能够直接操作函数b的变量，这个就是闭包的特性
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

综上所述，闭包就是函数a和变量x，也就是书上说的：
> 函数以及它所连接的周围作用域中的变量即为闭包

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

当然还有一个原因需要它，那就是**OO**，把上面发送验证码的代码改造一下变成面向对象风格
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
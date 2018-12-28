---
layout: post
category: "php"
title:  "php call_user_func"
---

```php
function foo($var1, $var2)
{
    return $var1 + $var2;
}

function increment(&$var)
{
    $var++;
}

class Test
{
    function sayHello($name) {
        echo "Hello, {$name}!\n";
    }
}

$a = 0;
$b = 2;

// 常规用法
$sum = call_user_func('foo', $a, $b);
echo $sum."\n";

// 匿名函数
call_user_func(function($i) {
    echo $i;
}, 666);

// 传入call_user_func()的参数不能为引用传递，下面这条语句会报错
$a1 = call_user_func('increment', $a);
echo $a1."\n";

// 用call_user_func_array来实现引用传递
call_user_func_array('increment', [&$a]);
echo $a."\n";
call_user_func_array('increment', [&$a]);
echo $a."\n";

// 调用类的方法
$test = new Test();
call_user_func([$test, 'sayHello'], 'pigfly');
call_user_func_array([$test, 'sayHello'], ['pigfly']);
```
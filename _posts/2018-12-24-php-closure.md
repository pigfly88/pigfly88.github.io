---
layout: post
category: "php"
title:  "php闭包"
---

```php
echo preg_replace_callback('/\s/', function($match) {
    return '';
}, 'a b  c')."<br />";

$sayHello = function($name) {
    echo "Hi~ {$name}<br />";
};
$name = 'pigfly';
$sayHello($name);

// 从父作用域继承变量
$sayHello2 = function () use ($name) {
    echo "Hello, {$name}!<br />";
};
$sayHello2();

$sayHello3 = function ($myname) use ($name, &$age) {
    $age++;
    echo "Hello, {$name}, My name is {$myname}, I'm {$age} years old.<br />";
};

$name = 'Allen';
$age = 18;
$sayHello3('pigfly');
$sayHello3('pigfly');
echo $age;
```
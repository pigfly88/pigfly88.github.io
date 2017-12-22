---
layout: post
category: "php"
title:  "PHP reference"
---

引用的含义
php的引用是用不同的名字访问同一个变量内容，并不像C的指针那样。在PHP 中，变量名和变量内容是不一样的， 因此同样的内容可以有不同的名字。最接近的比喻是 Unix 的文件名和文件本身——变量名是目录条目，而变量内容则是文件本身。引用可以被看作是 Unix 文件系统中的硬链接。

下面是变量引用：

$a = 1;
$b = &$a;
$b = 2;
var_dump($a, $b); // int(2) int(2)
$b = null;
var_dump($a, $b); // NULL NULL
引用能做什么
1.引用传递

<?php
function foo(&$var){
    $var++;
}
$a = 1;
foo($a); // 调用这个方法直接改变了$a的值，foo方法无需返回再赋值给$a
echo $a; // 这里会输出2
2.引用返回

将变量和函数返回值绑定

class foo {
    public $value = 42;

    public function &getValue() {
        return $this->value;
    }
}

$obj = new foo;
$myValue = &$obj->getValue(); // myvalue和$obj->value指向同一个变量
$obj->value = 2;
echo $myValue; // 输出2
$myValue = 3;
echo $obj->value; // 输出3
取消引用
当 unset 一个引用，只是断开了变量名和变量内容之间的绑定。这并不意味着变量内容被销毁了，例如：

$a = 1;
$b = &$a;
unset($a);
var_dump($a, $b); // 输出NULL int(1)
 
---
layout: post
category: "php"
title:  "PHP namespace"
---

### 介绍
印象中只有java代码才会用到一大堆的import，当初看到后一脸懵逼并对php心生自豪：还是我大php牛逼够简洁，殊不知php也有命名空间这一说，这些年用的越来越多。那么，为什么要搞那么麻烦呢？得写一大堆的use（神烦。。。一脸无奈），php手册给出了标准答案：

> 在PHP中，命名空间用来解决在编写类库或应用程序时创建可重用的代码如类或函数时碰到的两类问题
> - 用户编写的代码与PHP内部的类/函数/常量或第三方类/函数/常量之间的名字冲突
> - 为很长的标识符名称(通常是为了缓解第一类问题而定义的)创建一个别名（或简短）的名称，提高源代码的可读性。

好吧，换成二狗能理解的说法那就是：
- 解决命名冲突
- 重命名
### 举个栗子
```php
namespace my; //定义命名空间

//覆盖php类
class mysqli {
    public function query(){
        return 1;
    }
}

//覆盖php函数
function preg_replace_callback() {
    return 2;
}

//覆盖php常量
const PHP_SAPI = 3;

$a = new mysqli();
var_dump($a->query());

$b = preg_replace_callback();
var_dump($b);
var_dump(PHP_SAPI);
```
可以看到妥妥地返回了1,2,3：
```php
int(1) int(2) int(3)
```

那么问题来了，现在我要用php的mysqli怎么办？最前面加上\就好了：
```php
$a = new \mysqli;
```
我们在项目中遇到最多的情况是有两个同名的类库或方法而造成的冲突。假设有A,B两个第三方类库，它们都有Cache类，我要同时使用到他们两个：

├─application
│ ├─A
│ │ ├─Cache.php
│ ├─B
│ │ ├─Cache.php
│ ├─test.php

A/Cache.php:
```php
namespace A;
class Cache{
    function set(){
        return 'ok';
    }
}
```

```php
B/Cache.php:

namespace B;
class Cache{
    function set(){
        return 'success';
    }
}
```

test.php:
```php
require 'A/Cache.php';
require 'B/Cache.php';

$cache = new A\Cache();
var_dump($cache->set());

$cache = new B\Cache();
var_dump($cache->set());
```
返回：
```php
string(2) "ok" string(7) "success"
```
可以看到只要他两的命名空间不同，那么就可以正确调用到

### namespace和__NAMESPACE__
__NAMESPACE__返回当前命名空间字符串，namespace关键字可以用来显式访问当前命名空间或子命名空间中的元素
```php
$classname = __NAMESPACE__.'\mysqli';
$a = new $classname();
var_dump($a->query);

$a = new namespace\mysqli();
var_dump($a->query());
```

### use
use关键字就是用来指定使用哪个命名空间的，上面的例子我们没有使用到use是因为我们new的时候指定了路径，这样多麻烦呀，test.php改成使用use：
```php
use A\Cache;
require 'A/Cache.php';
require 'B/Cache.php';

$cache = new Cache(); //new A\Cache
var_dump($cache->set());

$cache = new B\Cache(); //new B\Cache
var_dump($cache->set());
```
这样每次new Cache就默认是实例化了A\Cache了，又可以早点回去和女票钻被窝了~

use as可以指定别名，当某个类库命名空间很长的时候就可以as一个短名称来偷个懒了，考虑类库代码如下：
```php
namespace Blah\Blah\Blah;
class CacheSomeThingImportingAndVeryDangerous{
    function set(){
        return 'success';
    }
}
```
![](https://images2017.cnblogs.com/blog/137801/201712/137801-20171201183850398-508938488.jpg)
天呐，这么长的方法名，整个人都不好了，use as一下，整个世界都安静了：
```php
use Blah\Blah\Blah\CacheSomeThingImportingAndVeryDangerous as Cache;
require 'B/CacheSomeThingImportingAndVeryDangerous.php';

$cache = new Cache();
var_dump($cache->set());
```
![](https://images2017.cnblogs.com/blog/137801/201712/137801-20171201183909305-1107812033.jpg)
以上！提前祝大家新年快乐！
---
layout: post
category: "php"
title:  "php自动加载和composer加载机制"
---

## 自动加载
PHP的自动加载以前使用__autoload方法实现的。

```php
function __autoload($class)
{
    include "Util/{$class}.php";
}

Foo::index(); // 加载Util/Foo.php
```

但是如果加载规则不一样会导致这个方法很长，所以后面都用spl_autoload_register()了，它可以注册多个自动加载方法，比如composer的自动加载：

```php
namespace Composer\Autoload;

class ClassLoader
{
    public function register($prepend = false)
    {
        spl_autoload_register(array($this, 'loadClass'), true, $prepend);
    }
}
```
 
现在加入有第三方组件有自己的自动加载机制，那么在注册一个就好了，比如Doctrine：
```php
<?php
namespace Doctrine\Common\Proxy;

class Autoloader
{
    public static function register($proxyDir, $proxyNamespace, $notFoundCallback = null)
    {
        $proxyNamespace = ltrim($proxyNamespace, '\\');

        $autoloader = function ($className) use ($proxyDir, $proxyNamespace, $notFoundCallback) {
            if (0 === strpos($className, $proxyNamespace)) {
                $file = Autoloader::resolveFile($proxyDir, $proxyNamespace, $className);

                if ($notFoundCallback && ! file_exists($file)) {
                    call_user_func($notFoundCallback, $proxyDir, $proxyNamespace, $className);
                }

                require $file;
            }
        };

        spl_autoload_register($autoloader);

        return $autoloader;
    }
}
```

## 自动加载的几种规范

### PSR-0

PSR-0会把下划线转换成目录分隔符，并且会把命名空间当成目录的一部分。

```php
// $baseDir/composer.json
"autoload": {
    "psr-0": {
        "Acme": "Util/" // 命名空间 => 目录
    }
}
```

```php
// vendor\composer\autoload_namespace.php
$vendorDir = dirname(dirname(__FILE__));
$baseDir = dirname($vendorDir);

return array(
    'ProxyManager\\' => array($vendorDir . '/ocramius/proxy-manager/src'),
    'Acme\\' => array($baseDir . '/Util'),
);
```

定义成以上规则时，当需要自动加载`Acme/Foo`的时候，查找路径是`Util/Acme/Foo.php`。

### PSR-4
PSR-4的命名空间不会当作目录的一部分，并且命名空间必须以`\\`结尾。

定义成以下规则时，当尝试加载`App/Entity/Product`的时候，查找路径是`src/Entity/Product.php`，`App`不会当成目录的一部分。

```php
// $baseDir/composer.json
"autoload": {
    "psr-4": {
        "App\\": "src/"
    }
}
```

```php
// vendor\composer\autoload_psr4.php
return array(
    'App\\' => array($baseDir . '/src'),
);
```

### classmap
一般用于没有命名空间的类加载。

```php
// vendor\symfony\polyfill-php73\composer.json
"autoload": {
    "classmap": [ "Resources/stubs" ]
},
```

那么执行conposer dump-autoload以后就会生成配置文件

```php
// vendor\composer\autoload_classmap.php
$vendorDir = dirname(dirname(__FILE__));
$baseDir = dirname($vendorDir);
return array(
    'JsonException' => $vendorDir . '/symfony/polyfill-php73/Resources/stubs/JsonException.php',
    'SqlFormatter' => $vendorDir . '/jdorn/sql-formatter/lib/SqlFormatter.php',
);
```

### files
用于加载一些方法。

```php
// vendor\symfony\polyfill-php73\composer.json
"autoload": {
    "files": [ "bootstrap.php" ]
},
```

那么执行conposer dump-autoload以后就会生成配置文件，然后在autoload入口文件会逐个require进来。

```php
// vendor\composer\autoload_files.php
$vendorDir = dirname(dirname(__FILE__));
$baseDir = dirname($vendorDir);
return array(
    '0d59ee240a4cd96ddbb4ff164fccea4d' => $vendorDir . '/symfony/polyfill-php73/bootstrap.php',
);
```
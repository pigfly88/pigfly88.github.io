---
layout: post
category: "php"
title:  "设计模式-工厂模式"
---

## 抽象工厂
Provide an interface for creating families of related or dependent objects without specifying their concrete classes.
存在一系列相关的类，提供一个接口，可以在不指定具体类名的情况下，来创建拥有相同
https://github.com/pear/Mail/blob/master/Mail.php
```php
class Mail
{
    public static function factory($driver, $params = array())
    {
        $driver = strtolower($driver);
        @include_once 'Mail/' . $driver . '.php';
        $class = 'Mail_' . $driver;
        if (class_exists($class)) {
            $mailer = new $class($params);
            return $mailer;
        } else {
            return PEAR::raiseError('Unable to find class for driver ' . $driver);
        }
    }
}
```

https://github.com/pear/MDB2/blob/master/MDB2.php

```php
class MDB2
{
    static function factory($dsn, $options = false)
    {
        $dsninfo = MDB2::parseDSN($dsn);
        if (empty($dsninfo['phptype'])) {
            $err = MDB2::raiseError(MDB2_ERROR_NOT_FOUND,
                null, null, 'no RDBMS driver specified');
            return $err;
        }
        $class_name = 'MDB2_Driver_'.$dsninfo['phptype'];
        $debug = (!empty($options['debug']));
        $err = MDB2::loadClass($class_name, $debug);
        if (MDB2::isError($err)) {
            return $err;
        }
        $db = new $class_name();
        $db->setDSN($dsninfo);
        $err = MDB2::setOptions($db, $options);
        if (MDB2::isError($err)) {
            return $err;
        }
        return $db;
    }
}
```

### 参考资料
- [Docker 官方文档](https://docs.docker.com/get-started/)
- [Docker 入门教程](http://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html)
- [Docker — 从入门到实践](https://docker_practice.gitee.io/)
- [Docker for beginners](https://github.com/docker/labs/tree/master/beginner)
- [Setting up PHP, PHP-FPM and NGINX for local development on Docker](https://www.pascallandau.com/blog/php-php-fpm-and-nginx-on-docker-in-windows-10/)
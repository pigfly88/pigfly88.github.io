---
layout: post
category: "php"
title:  "设计模式-工厂模式"
---


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

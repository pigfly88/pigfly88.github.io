---
layout: post
category: "php"
title:  "依赖注入"
---

**依赖注入就是把所依赖的对象先在外面创建好了，再通过构造函数或者set方法注入进来。**

```php
// 常规做法
class User
{
	protected $logger;
	
    public function __construct()
    {
        $this->logger = new Logger();
    }
}


// 依赖注入
class User
{
    protected $logger;
	
    public function __construct($logger)
    {
        $this->logger = $logger;
    }
}

$logger = new Logger();
$user = new User($logger);
```
---
layout: post
category: "php"
title:  "设计模式-工厂模式"
---

### 简单工厂

出现背景：当项目中存在很多相同的做特定实例化的代码，会造成难以修改。

解决方法：把创建具体对象的代码封装到工厂里面，由工厂来完成实例化。

```php
class ConnectionFactory
{
    protected function createConnection($driver, $connection, $database, $prefix = '', array $config = [])
    {
        switch ($driver) {
            case 'mysql':
                return new MySqlConnection($connection, $database, $prefix, $config);
            case 'pgsql':
                return new PostgresConnection($connection, $database, $prefix, $config);
            case 'sqlite':
                return new SQLiteConnection($connection, $database, $prefix, $config);
            case 'sqlsrv':
                return new SqlServerConnection($connection, $database, $prefix, $config);
        }

        throw new InvalidArgumentException("Unsupported driver [{$driver}].");
    }
}
```

### 工厂方法

### 抽象工厂

### 观察者

出现背景：当发生某一事件的时候需要一系列的对象作出响应，比如当下单成功的时候需要给用户发送短信和发送邮件。

```php
class OrderService implements SplSubject
{
    private $observers;
    
    public function __construct()
    {
        $this->observers = new SplObjectStorage();
    }
    
    public function attach(SplObserver $observer)
    {
        $this->observers->attach($observer);
    }
    
    public function detach(SplObserver $observer)
    {
        $this->observers->detach($observer);
    }
    
    public function notify()
    {
        foreach ($this->observers as $observer) {
            $observer->update($this);
        }
    }
    
    public function pay()
    {
        $this->notify();
    }
}

class SmsService implements SplObserver
{
    public function update(SplSubject $subject)
    {
        echo "send sms\n";
    }
}

class EmailService implements SplObserver
{
    public function update(SplSubject $subject)
    {
        echo "send email\n";
    }
}

$orderService = new OrderService();
$orderService->attach(new SmsService());
$orderService->attach(new EmailService());
$orderService->pay();
```

### 参考资料

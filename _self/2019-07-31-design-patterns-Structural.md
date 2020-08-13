---
layout: post
category: "php"
title:  "设计模式"
---

### 单例

出现背景：有些对象只需要实例化一次。

解决方案：实例化之后保存起来，禁止new(), __clone(), __wakeup()，保证只能实例化一次

举个例子：数据库连接

以下代码摘自 Laravel：

```php
namespace Illuminate\Database;

class DatabaseManager implements ConnectionResolverInterface
{
	/**
     * The active connection instances.
     *
     * @var array
     */
    protected $connections = [];
	
	/**
	 * Get a database connection instance.
	 *
	 * @param  string|null  $name
	 * @return \Illuminate\Database\Connection
	 */
	public function connection($name = null)
	{
		[$database, $type] = $this->parseConnectionName($name);

		$name = $name ?: $database;

		if (! isset($this->connections[$name])) {
			$this->connections[$name] = $this->configure(
				$this->makeConnection($database), $type
			);
		}

		return $this->connections[$name];
	}
}
```


### 原型

出现背景：当需要复制一个一模一样的对象，并且不希望影响到原来的对象时。

解决方案：复制现有对象的所有属性来实例化新对象，从而创建独立的克隆。当新对象的构造效率低下时，此做法特别有用。

举个例子：

以下代码摘自 Symfony：

```php
namespace Symfony\Component\HttpFoundation;

class Request
{
	/**
     * Clones a request and overrides some of its parameters.
     *
     * @param array $query      The GET parameters
     * @param array $request    The POST parameters
     * @param array $attributes The request attributes (parameters parsed from the PATH_INFO, ...)
     * @param array $cookies    The COOKIE parameters
     * @param array $files      The FILES parameters
     * @param array $server     The SERVER parameters
     *
     * @return static
     */
    public function duplicate(array $query = null, array $request = null, array $attributes = null, array $cookies = null, array $files = null, array $server = null)
    {
        $dup = clone $this;
        if (null !== $query) {
            $dup->query = new InputBag($query);
        }
        if (null !== $request) {
            $dup->request = new ParameterBag($request);
        }
        if (null !== $attributes) {
            $dup->attributes = new ParameterBag($attributes);
        }
        if (null !== $cookies) {
            $dup->cookies = new InputBag($cookies);
        }
        if (null !== $files) {
            $dup->files = new FileBag($files);
        }
        if (null !== $server) {
            $dup->server = new ServerBag($server);
            $dup->headers = new HeaderBag($dup->server->getHeaders());
        }
        $dup->languages = null;
        $dup->charsets = null;
        $dup->encodings = null;
        $dup->acceptableContentTypes = null;
        $dup->pathInfo = null;
        $dup->requestUri = null;
        $dup->baseUrl = null;
        $dup->basePath = null;
        $dup->method = null;
        $dup->format = null;

        if (!$dup->get('_format') && $this->get('_format')) {
            $dup->attributes->set('_format', $this->get('_format'));
        }

        if (!$dup->getRequestFormat(null)) {
            $dup->setRequestFormat($this->getRequestFormat(null));
        }

        return $dup;
    }
}
```
	
### 简单工厂

出现背景：当项目中存在很多相同的做特定实例化的代码，会造成难以修改。

解决方案：把创建具体对象的代码封装到工厂里面，由工厂来完成实例化。

举个例子：数据库连接实例化的时候，根据驱动类型来做实例化。

优点：1、一个调用者想创建一个对象，只要知道其名称就可以了。2、屏蔽产品的具体实现，调用者只关心产品的接口。

缺点：1、新增一种类型要多加一个case，不符合开闭原则。

以下代码摘自 Laravel：
```php
namespace Illuminate\Database\Connectors;

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

解决方案：定义一个创建对象的接口，但让实现这个接口的类来决定实例化哪个类，工厂方法让类的实例化推迟到子类中进行。

### 抽象工厂

### 工厂模式总结

> 当创建逻辑比较复杂，是一个“大工程”的时候，我们就考虑使用工厂模式，封装对象的创建过程，将对象的创建和使用相分离。何为创建逻辑比较复杂呢？我总结了下面两种情况。

	- 第一种情况：类似规则配置解析的例子，代码中存在 if-else 分支判断，动态地根据不同的类型创建不同的对象。针对这种情况，我们就考虑使用工厂模式，将这一大坨 if-else 创建对象的代码抽离出来，放到工厂类中。

	- 还有一种情况，尽管我们不需要根据不同的类型创建不同的对象，但是，单个对象本身的创建过程比较复杂，比如前面提到的要组合其他类对象，做各种初始化操作。在这种情况下，我们也可以考虑使用工厂模式，将对象的创建过程封装到工厂类中。

对于第一种情况，当每个对象的创建逻辑都比较简单的时候，我推荐使用简单工厂模式，将多个对象的创建逻辑放到一个工厂类中。当每个对象的创建逻辑都比较复杂的时候，为了避免设计一个过于庞大的简单工厂类，我推荐使用工厂方法模式，将创建逻辑拆分得更细，每个对象的创建逻辑独立到各自的工厂类中。同理，对于第二种情况，因为单个对象本身的创建逻辑就比较复杂，所以，我建议使用工厂方法模式。

除了刚刚提到的这几种情况之外，如果创建对象的逻辑并不复杂，那我们就直接通过 new 来创建对象就可以了，不需要使用工厂模式。

现在，我们上升一个思维层面来看工厂模式，它的作用无外乎下面这四个。这也是判断要不要使用工厂模式的最本质的参考标准。

	- 封装变化：创建逻辑有可能变化，封装成工厂类之后，创建逻辑的变更对调用者透明。

	- 代码复用：创建代码抽离到独立的工厂类之后可以复用。

	- 隔离复杂性：封装复杂的创建逻辑，调用者无需了解如何创建对象。

	- 控制复杂度：将创建代码抽离出来，让原本的函数或类职责更单一，代码更简洁。

### 观察者

出现背景：当发生某一事件的时候需要一系列的对象作出响应。

解决方案：将需要作出反应的对象加入到观察者列表，当发生特定事件的时候循环调用观察者的响应事件。

举个例子：当下单成功的时候需要给用户发送短信和发送邮件。

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

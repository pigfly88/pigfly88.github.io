---
layout: post
category: "php"
title:  "Laravel的生命周期"
---

### 核心概念

- Service Container：服务容器，我们需要用到的服务由它来管理，比如日志服务、数据库服务等等
- Service Providers：服务提供者，比如日志服务，数据库服务等等
- Facades：门面
- Contracts：合约，指的是接口

服务容器帮助我们实现依赖注入，我们可以很方便的使用服务，而服务提供者里面定义了依赖注入的绑定关系。

### 入口

从`public/index.php`这个入口文件开始，Laravel开启了它的生命之旅：

```
// 1. 记录开始时间
define('LARAVEL_START', microtime(true));

// 2. composer自动加载
require __DIR__.'/../vendor/autoload.php';

// 3. 创建一个app容器并注册核心服务
$app = require_once __DIR__.'/../bootstrap/app.php';

// 4. 实例化一个HTTP内核
$kernel = $app->make(Illuminate\Contracts\Http\Kernel::class);

// 5. 处理请求，得到响应内容
$response = $kernel->handle(
    $request = Illuminate\Http\Request::capture()
);

// 6. 发送响应内容
$response->send();

// 7. 善后处理
$kernel->terminate($request, $response);
```

前面两步都不说了，composer机制不了解的可以自行搜索，来看看第3步的时候，创建app容器的代码：

```php
// bootstrap/app.php

$app = new Illuminate\Foundation\Application(
    $_ENV['APP_BASE_PATH'] ?? dirname(__DIR__)
);

$app->singleton(
    Illuminate\Contracts\Http\Kernel::class, // 接口，是抽象的
    App\Http\Kernel::class // 实现接口的类，是具体的
);

$app->singleton(
    Illuminate\Contracts\Console\Kernel::class,
    App\Console\Kernel::class
);

$app->singleton(
    Illuminate\Contracts\Debug\ExceptionHandler::class,
    App\Exceptions\Handler::class
);

return $app;
```

```php
namespace Illuminate\Foundation;

class Application extends Container implements ApplicationContract, CachesConfiguration, CachesRoutes, HttpKernelInterface
{
    public function __construct($basePath = null)
    {
        if ($basePath) {
            $this->setBasePath($basePath);
        }

        $this->registerBaseBindings();
        $this->registerBaseServiceProviders();
        $this->registerCoreContainerAliases();
    }
}
```

我们来看看app容器的构造函数，Foundation 是基类的意思，`$app`容器的构造函数设定了基础路径，注册基础绑定信息，注册基础`Service Provider`和 Aliases，而做这些工作的意义就是可以帮助我们更方便的访问和获取项目目录中的文件信息，加载配置文件，加载自定义的类，加载Service，aliases 定义了如何把开发框架中最基础的服务注册到容器中，看了这部分代码，咱们就知道为啥 app('files') 和 app ('Illuminate\Filesystem\Filesystem') 这两种方法都可以拿到 FileSystem 服务的原因了。

我们再来看app容器实例化以后，绑定了 HTTP Kernal 和 Console Kernel，分别用来处理 HTTP 网络请求和 CLI 请求（ 执行 php artisan 相关命令请求），还绑定了用来处理应用运行异常的调试处理器。

`$app->singleton(a,b)`这是做了一个依赖注入的绑定，并且指明了实例化b的方式是单例，绑定是绑定什么？就是用到a接口的时候就实例化b出来，这个b类是实现了a接口的。为什么要绑定？因为[依赖注入](http://blog.phpzendo.com/?p=313)提倡我们使用面向接口/抽象的思维来带入对象，而不是依赖于具体的某个类，依赖接口/抽象可以达到**控制反转**。

这是把HTTP内核接口绑定到一个实现了它的接口的具体的类上面，这么做有什么效果？

我们看到`public/index.php`这儿第4步实例化一个HTTP内核的时候并不是直接`new App\Http\Kernel()`，而是用`$app->make(Illuminate\Contracts\Http\Kernel::class)`，我们看到这里传入的是一个接口，这就是面向接口，而前面singleton已经把这个接口绑定到具体的类，所以容器一看到这个接口的时候它就很清楚的知道要去实例化一个什么类了。

好了，上面说了一大堆都是说的[依赖注入](http://blog.phpzendo.com/?p=313)，其实容器就是帮我们做这个的。
---
layout: post
category: "php"
title:  "ThinkPHP5源码解析(2)控制器"
---

入口文件index.php

```php
// 定义应用目录
define('APP_PATH', __DIR__ . '/../application/');
// 加载框架引导文件
require __DIR__ . '/../thinkphp/start.php';
```

引导文件start.php
```php
namespace think;

// 加载基础文件
require __DIR__ . '/base.php';
// 执行应用
App::run()->send();
```

基础文件base.php
```php
defined('THINK_PATH') or define('THINK_PATH', __DIR__ . DS);
define('LIB_PATH', THINK_PATH . 'library' . DS);
define('CORE_PATH', LIB_PATH . 'think' . DS);
// 此处省略一堆的define...

// 载入Loader类
require CORE_PATH . 'Loader.php';
// 注册自动加载
\think\Loader::register();
```

think\App::run
```php
public static function run(Request $request = null)
{
	is_null($request) && $request = Request::instance();
	$config = self::initCommon();
	
	// 获取应用调度信息
	$dispatch = self::routeCheck($request, $config);
	// 记录当前调度信息
	$request->dispatch($dispatch);
	// 执行调度
	$data = self::exec($dispatch, $config);

	// 输出数据到客户端
	if ($data instanceof Response) {
		$response = $data;
	} elseif (!is_null($data)) {
		// 默认自动识别响应输出类型
		$isAjax   = $request->isAjax();
		$type     = $isAjax ? Config::get('default_ajax_return') : Config::get('default_return_type');
		$response = Response::create($data, $type);
	} else {
		$response = Response::create();
	}

	return $response;
}
```
routeCheck拿到$_SERVER['PATH_INFO']获取控制器和操作名，返回格式：
```php
array(2) {
  ["type"] => string(6) "module"
  ["module"] => array(3) {
    [0] => NULL
    [1] => string(5) "index"
    [2] => string(5) "hello"
  }
}
```

exec()调用module()通过反射来实例化控制器和执行操作：
```php
protected static function exec($dispatch, $config)
{
	$data = self::module($dispatch['module'], $config);			
	return $data;
}

/**
 * 执行模块
 * @access public
 * @param array $result 模块/控制器/操作
 * @param array $config 配置参数
 * @param bool  $convert 是否自动转换控制器和操作名
 * @return mixed
 */
public static function module($result, $config)
{
	// 获取控制器名
	$controller = strip_tags($result[1] ?: $config['default_controller']);
	// 获取操作名
	$action = strip_tags($result[2] ?: $config['default_action']);
	// 实例化控制器
	$instance = self::invokeClass($controller);	
	// 执行控制器方法
	return self::invokeMethod([$instance, $action]);
}

/**
 * 调用反射执行类的实例化 支持依赖注入
 * @access public
 * @param string    $class 类名
 * @param array     $vars  变量
 * @return mixed
 */
public static function invokeClass($class, $vars = [])
{
	$reflect     = new \ReflectionClass($class);
	$constructor = $reflect->getConstructor();
	if ($constructor) {
		$args = self::bindParams($constructor, $vars);
	} else {
		$args = [];
	}
	return $reflect->newInstanceArgs($args);
}

/**
 * 调用反射执行类的方法 支持参数绑定
 * @access public
 * @param string|array $method 方法
 * @param array        $vars   变量
 * @return mixed
 */
public static function invokeMethod($method, $vars = [])
{
	if (is_array($method)) {
		$class   = is_object($method[0]) ? $method[0] : self::invokeClass($method[0]);
		$reflect = new \ReflectionMethod($class, $method[1]);
	} else {
		// 静态方法
		$reflect = new \ReflectionMethod($method);
	}
	$args = self::bindParams($reflect, $vars);

	return $reflect->invokeArgs(isset($class) ? $class : null, $args);
}
```

最终run()获得Response对象，调用Response->send()方法输出数据

### 结论
$_SERVER['PATH_INFO']获得模块/控制器/操作，通过反射实例化控制器并执行操作

### 疑问
为什么没有直接用autoload来调度控制器呢？

### 缺点
不支持单模块和多模块并存，配置文件需要指定是否多模块，我觉得可以做成共存，这样很容易实现控制器的复用，比如：
- Application/controller/Index.php
- Application/a/controller/Index.php
当url为：/a/index/index，优先执行a模块Application/a/controller/Index的index方法，如果a模块不存在这个方法，调用外围的Application/controller/Index下的index方法
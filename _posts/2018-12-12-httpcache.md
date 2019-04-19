---
layout: post
category: "网络"
title:  "HTTP缓存"
---

缓存在web环境各个环节都有实现，有CPU缓存、文件缓存、程序的Opcode缓存（APC,eAccelerator）、内存缓存（Memcached,Redis）、代理服务器(Nginx,Squid)、数据库的查询缓存、基于HTTP的客户端缓存。其中HTTP缓存是离用户最近的缓存，访问最快，合理使用可以加快数据加载速度、减少服务器的开销。

HTTP缓存通过设置一些头加以控制，有一部分是控制要不要缓存、怎么缓存、以及缓存多久的，还有一部分是决定缓存过期以后怎么处理的。下面只列出最主要的：

- 缓存控制
	- Cache-Control
		- public：公共，可以被客户端和代理服务器（如Nginx、Squid）缓存
		- private：私有，只能被客户端缓存
		- no-store：不缓存
		- no-cache：每次请求都要发起请求询问服务器缓存是否可用，配合ETag或者Last-Modified
		- must-revalidate：因为max-stale允许客户端使用过期的缓存，通过指定这个头，对于过期的缓存，会强制跟服务器重新比对缓存是否可用
	- Pragma
	 - Pragma相比Cache-Control是不支持响应头的，通常是为了向后兼容HTTP/1.0

- 缓存校验
	- 过期
		- max-age
			- 缓存时间，距离请求发起的时间的秒数，设置到Cache-Control头里面
		- Expires
			- 过期时间(GMT格式)，和max-age类似，但是Expires受客户端时间影响，是HTTP/1.0的标准，max-age是它的改良版，优先级为：max-age -> Expires
	- 比对
		- Last-Modified
			- 服务端在响应头里面设置此项来表明修改时间(GMT格式)，客户端会在下次请求时自动带上If-Modified-Since: <last_modified_value>，以此来比对缓存是否有更新
		- ETag(Entity Tags)
			- 依赖Last-Modified来检查缓存有缺陷，比如文件的修改时间变了但是内容没有变，又比如它只能精确到秒，而ETag是比对内容，可以理解为md5值，服务端响应ETag后客户端会在下次请求时自动带上If-None-Match: <etag_value>。ETag比Last-Modified开销大，如果可以用Last-Modified尽量用Last-Modified

### 实例
缓存一分钟，一分钟内直接读取本地缓存，一分钟后重新请求服务器：

```
Cache-Control: max-age=60, must-revalidate
```

上面这种方式一旦缓存过期就会重新请求服务器返回最新的内容，如果内容并没有变化，那不是白传了？有没有办法在缓存过期以后判断一下如果内容没有改变则继续用本地的缓存呢？很简单！ETag可以帮到你，有效期内直接读取本地缓存，过期后跟服务器比对ETag，相同则服务器会返回304表示缓存还可以继续用，而不返回实际内容，节约了时间和带宽。

缓存一分钟，一分钟内直接读取本地缓存，一分钟以后跟服务器比对ETag，如果ETag没有变化，那么接下来的一分钟内直接读取本地缓存：

```
Cache-Control: max-age=60, must-revalidate
ETag: abc
```

每次都要跟服务器比对ETag是否相同，适合更新稍微比较频繁并且需要及时显示最新内容的资源：

```
Cache-Control: no-cache
ETag: abc
```

不缓存：

```
Cache-Control: no-store
```

### 一图胜千言
![http-cache](/images/http-cache.png)

### API缓存
我们一般会在Nginx上配置静态文件的缓存，而常常忽略了动态页面和API的缓存，其实它们也是可以设置缓存的，在代码里面实现更加灵活，以PHP代码为例，新建一个文件命名为cache.php：

```php
header('Cache-Control: public, max-age=60, must-revalidate');
$time = date('H:i:s');
$data = 666;
$etag = md5($data);

if($_SERVER['HTTP_IF_NONE_MATCH'] == $etag){
    header("HTTP/1.1 304 Not Modified");
}else{
    header('ETag: '.$etag);
}
echo $time.'|'.$data;
```

上面的代码设置缓存一分钟，过期之后客户端需要和服务端通过比对ETag来确认缓存是否可以继续使用。现在可以一直点击刷新按钮看看效果，奇怪，怎么不是直接读本地缓存，老是请求到服务器返回304了呢？

这是因为浏览器做了手脚，点击浏览器的刷新按钮后，请求头会加上max-age=0，表示客户端放弃读缓存，如果想看200 from disk cache的效果，需要一个页面来做个跳转：

```html
<!DOCTYPE html>
<html lang="zh">
<head>
  <meta charset="utf-8">
</head>
<body>
	<p><a href="./cache.php">这个页面被缓存了，打开看看吧。</a></p>
</body>
</html>
```

### PHP中一些成熟的类库
[laravel-httpcache](https://github.com/barryvdh/laravel-httpcache)
[symfony-httpcache](https://github.com/symfony/symfony/tree/master/src/Symfony/Component/HttpKernel/HttpCache)
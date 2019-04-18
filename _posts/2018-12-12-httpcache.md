---
layout: post
category: "网络"
title:  "HTTP缓存"
---

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
			- 缓存时间，距离请求发起的时间的秒数
		- Expires
			- 和max-age类似，但是Expires受客户端时间影响，是HTTP/1.0的标准，max-age是它的改良版，优先级为：max-age -> Expires
	- 比对
		- Last-Modified
			- 资源修改时间，与客户端的If-Modified-Since配合使用
		- ETag(Entity Tags)
			- 依赖Last-Modified来检测资源是否修改会有缺陷，比如文件的修改时间变了但是内容没有变，ETag是比对资源的内容，可以理解为资源的md5值，用来比对缓存内容是否发生改变，与客户端的If-None-Match配合使用

### 实例
缓存一分钟，一分钟内直接读取本地缓存，一分钟后重新请求服务器：

```http
Cache-Control: max-age=60, must-revalidate
```

上面这种方式一旦缓存过期就会重新请求服务器返回最新的内容，如果内容并没有变化，那不是白传了？有没有办法在缓存过期以后判断一下如果内容没有改变则继续用本地的缓存呢？很简单！ETag可以帮到你，有效期内直接读取本地缓存，过期后跟服务器比对ETag，相同则服务器会返回304表示缓存还可以继续用，而不返回实际内容，节约了时间和带宽。

缓存一分钟，一分钟内直接读取本地缓存，一分钟以后跟服务器比对ETag，如果ETag没有变化，那么接下来的一分钟内直接读取本地缓存：

```http
Cache-Control: max-age=60, must-revalidate

ETag: abc
```

每次都要跟服务器比对ETag是否相同，适合更新稍微比较频繁并且需要及时显示最新内容的资源：

```http
Cache-Control: no-cache

ETag: abc
```

不缓存：

```http
Cache-Control: no-store
```

### 一图胜千言
![http-cache](/images/http-cache.png)
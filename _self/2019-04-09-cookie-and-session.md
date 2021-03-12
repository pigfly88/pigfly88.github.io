### cookie
HTTP是无状态的，如果要把一些数据保存到客户端，然后服务端可以很轻松地拿到这些数据怎么办？比如在用户未登录的状态下记录购物车里面的商品，这时候cookie就派上用场了，用户点击加入购物车时，客户端利用JS将商品保存到cookie中（当然也可以ajax请求交给php调用一个setcookie，其实就是在HTTP头里面加上set-cookie，注意php并没有直接去操作cookie，而是依赖HTTP协议，客户端收到set-cookie的操作之后进行cookie的设置），然后客户端下一次请求服务端的时候会自动把cookie带上。

由于cookie在每次请求都会带上（可以设置path，只有在该路径下的请求才发送），在一定程度上造成网络开销的浪费，相比之下Local Storage会是一个比较好的方案。

### session
cookie是保存在客户端的，而session是通过一个特殊的cookie(PHPSESSID)来搭桥，把数据保存到服务端。这个cookie就相当于一个key，服务端通过这个key去找到对应的session。

### cookie 和 session 的区别与联系

- cookie保存到客户端，session的内容保存到服务端
- cookie有容量（4K左右）和个数（50个左右）的限制（取决于浏览器），session没有大小限制
- session一般是借助cookie的方式来实现的：在客户端存一个PHPSESSID的cookie。当然，如果客户端禁用cookie的话，还可以用url query的方式


### php session 配置

```ini
session.save_handler = files # 默认是文件方式保存在磁盘上，也支持memcached
session.gc_probability = 1
session.gc_divisor = 1000 #session回收概率 session.gc_probability/session.gc_divisor: 1/1000
session.gc_maxlifetime = 1440 #过期时间，默认24分钟，如果触发了session回收，发现session的最后修改时间是24分钟以前了，那么会删除这个session文件
session.save_path = /tmp # 保存路径，如果是memcached的话可以写成memcached地址，比如 tcp://127.0.0.1:11211
```

### php session 原理

1. session_start

会判断当前是否有$_COOKIE[session_name()]，默认的session_name是PHPSESSID。

如果不存在会生成一个session_id，然后把生成的session_id作为cookie的值传递到客户端。相当于执行了下面cookie操作

```php
setcookie(session_name(),
    session_id(),
    session.cookie_lifetime,//默认0
    session.cookie_path,//默认'/'当前程序跟目录下都有效
    session.cookie_domain,//默认为空
)
```

如果存在那么去session.save_path指定的文件夹里去找名字为'SESS_'.session_id()的文件。读取文件的内容反序列化，然后放到$_SESSION中。

1. 为$_SESSION赋值

```php
$_SESSION['test'] ='blah';
```
那么这个$_SESSION只会维护在内存中，当脚本执行结束的时候，再把session内容写入到session_id指定的文件中。

1. 删除session

```php
setcookie(session_name(),session_id(),time()-86400); //退出登录前执行
usset($_SESSION); //这会删除所有的$_SESSION数据，刷新后，有COOKIE传过来，但是没有数据。
session_destroy(); //这个作用更彻底，删除$_SESSION 删除session文件，和session_id
```

### 如何设置一个严格1分钟过期的session
常见做法是通过设置PHPSESSID这个cookie的过期时间：
```php
session_set_cookie_params(60, '/', 'test.com');
// 或者：
setcookie(session_name(), session_id(), time()+60);
```
这么设置以后PHPSESSID这个cookie在1分钟以后就会过期，没有了它自然也就没办法去服务器找到对应的session文件。但如果在过期之前把它记下来，等过期以后再手动填上，这样请求到服务器这边还是可以拿到对应的session，因为服务器上的session文件很有可能并没有被删除，session回收是有一定概率的，如果概率调得太大会影响性能。

我们可以保存一个过期时间戳来判断，获取session的时候先判断是否过期：
```php
// 登录成功设置session
session_set_cookie_params(60, '/', 'test.com'); // 这个也加上，双保险
session_start();
$_SESSION['login'] = 1;
$_SESSION['login_expire'] = time() + 60;

// 另一个页面，检查登录状态
session_start();
if($_SESSION['login']){
    if(time() > $_SESSION['login_expire']) {
        echo '登录状态已失效，请重新登录';
    }else{
        echo '已登录';
    }
}else{
    echo '未登录';
}
```

### 单点登录

### 分布式系统如何保存session

### 验证码 session 问题

### 安全的 session_id

### 记住密码的正确姿势
参考左耳朵耗子的[你会做WEB上的用户登录功能吗？](https://coolshell.cn/articles/5353.html)，还可以看看symfony是怎么做账号安全这一块的：[Security](https://symfony.com/doc/current/security.html)。



## 参考资料
- [PHP session的工作原理](http://www.nowamagic.net/librarys/veda/detail/358)
- [彻底理解PHP的SESSION机制](https://www.cnblogs.com/acpp/archive/2011/06/10/2077592.html)


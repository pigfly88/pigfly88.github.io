### Cookie
HTTP是无状态的，如果要把一些数据保存到客户端，然后服务端可以很轻松地拿到这些数据怎么办？比如在用户未登录的状态下记录购物车里面的商品，这时候Cookie就派上用场了，用户点击加入购物车时，客户端利用JS将商品保存到Cookie中（当然也可以ajax请求交给php调用一个setcookie，其实就是在HTTP头里面加上set-cookie，注意php并没有直接去操作Cookie，而是依赖HTTP协议，客户端收到set-cookie的操作之后进行Cookie的设置），然后客户端下一次请求服务端的时候会自动把Cookie带上。

由于Cookie在每次请求都会带上（可以设置path，只有在该路径下的请求才发送），在一定程度上造成网络开销的浪费，相比之下Local Storage会是一个比较好的方案。

### Session
Cookie是保存在客户端的，而Session是借助一个特殊的Cookie把数据保存到服务端，在PHP中，这个特殊的Cookie叫做PHPSESSID。

### 如何设置一个严格1分钟过期的Session
常见做法是通过设置PHPSESSID这个Cookie的过期时间：
```php
session_set_cookie_params(60, '/', 'test.com');
// 或者：
setcookie(session_name(), session_id(), time()+60);
```
这么设置以后PHPSESSID这个Cookie在1分钟以后就会过期，没有了它自然也就没办法去服务器找到对应的Session文件。但如果在过期之前把它记下来，等过期以后再手动填上，这样请求到服务器这边还是可以拿到对应的Session，因为服务器上的Session文件很有可能并没有被删除，Session回收是有一定概率的，如果概率调得太大会影响性能。

我们可以保存一个过期时间戳来判断，获取Session的时候先判断是否过期：
```php
// 登录成功设置Session
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
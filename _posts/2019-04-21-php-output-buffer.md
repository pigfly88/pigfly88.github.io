---
layout: post
category: "php"
title:  "PHP输出缓冲"
---

对于PHP输出的内容（echo、print等等），会先放到输出缓冲，当缓冲区写满了或者程序执行完毕了才会转移到下一层，这么做可以合并数据减少传输次数。


![ob-main](/images/ob-main.png)

PHP涉及到的输出缓冲有三层。

- 用户输出缓冲层：ob_start()就是在这个层里面新建了一个缓冲区
- 默认输出缓冲层：PHP对于所有输出使用的默认的一个缓冲层
- SAPI缓冲层：比如Apache发送内容给浏览器之前，维护的一个缓冲层

上面的层级关系从上到下，所有输出会依次按顺序进入，只有当前面的缓冲层满了，或者手动冲刷，又或者是代码运行完了，才会转移到下一层。

### 什么时候冲刷

在非CLI环境下，PHP的默认输出缓冲是默认开启的，缓冲大小是4096字节，这也就是为什么下面的代码，在我们用浏览器打开页面的时候要等两秒才能看到666。

index.php：
```
<?php
echo 666;
sleep(2);
```

而同样的代码在CLI环境下截然不同，你可以在命令行执行```php -f index.php```看看，马上会输出666。这是因为在CLI的SAPI下，PHP默认不使用输出缓冲（output_buffering = Off，implicit_flush = On），所有输出会立即转移到下一层。

假如我们把PHP的配置改一下：
```shell
php -d output_buffering=2 -d implicit_flush=1 -S 127.0.0.1:8080
```

这时候我们用浏览器打开127.0.0.1:8080，666马上就显示出来了，因为我们把PHP的默认输出缓冲区设置为2个字节了，666是3个字节不够放了，这时候只能移交到下一层的SAPI缓冲层了，而又因为我们设置了implicit_flush=1，SAPI的缓冲层也立马将输出发给浏览器了。

直接修改PHP配置不是一个明智的选择，毕竟不是所有脚本要处理的情况都是一模一样的，这时候我们可以通过PHP提供的ob系列函数来控制：
```php
echo 666;
ob_flush();
flush();
sleep(2);
```

```shell
php -S 127.0.0.1:8080
```
浏览器打开127.0.0.1:8080，PHP以CGI模式运行，不改变PHP配置的情况下，666立马显示了出来。

我们调用了ob_flush()和flush()，这两个函数是干嘛的？怎么感觉他两有点眼熟，好像是经常在一起的。

### ob_flush()和flush()
ob_flush()是冲刷PHP的默认输出缓冲层，flush()是冲刷SAPI缓冲层。

我们输出的666并没有让PHP的默认输出缓冲层溢出，这样就到不了SAPI缓冲层也就更谈不上给浏览器了。调用ob_flush()之后会冲刷到SAPI缓冲层，而flush()则是告诉Apache：你也立马把内容发给浏览器吧。Apache也乖乖地从SAPI缓冲层拿到内容直接发给浏览器了。

### 合理flush可以提高用户体验
正常情况下，4096字节是一个不错的值，设置得太小，SAPI和客户端的传输次数就会变多，但假如我们的页面有些模块加载起来特别耗时，我们可以先把那些能够快速拿到的内容先冲刷给浏览器，这样用户就不会等半天还是个白屏状态。
```php
echo 666;
ob_flush();
flush();

// 耗时任务
sleep(2);
$a = 1;
echo $a;
```
其中echo 666那块可以自己脑补，比如可以是个网站头部，有登录注册按钮，有导航之类的，这些内容很快就拿到了，用户不用等你写的很烂的sql查询完了才能点击美女频道。

### 用户缓冲层
上面说到的都是PHP的默认缓冲层，在它之上还可以有用户缓冲层，我们可以通过ob_start()创建用户缓冲区，用户缓冲区可以创建多个，最后创建的会在最上面，这样层层堆叠下来，如果没有关闭，输出会依次进入这些缓冲区，然后到PHP默认缓冲层再到SAPI缓冲层。

```php
echo ob_get_level()."\n"; // 1，PHP默认缓冲层
ob_start(); // 新建一个用户缓冲区，假设它叫UC1
echo ob_get_level()."\n"; // 因为上面新建了一个缓冲区，所以这里会显示2
echo "abc\n";
ob_end_flush(); // 冲刷并关闭最顶端的缓冲区，这里是UC1
echo ob_get_level()."\n"; // 1，UC1已经被关闭了
ob_flush(); // 冲刷PHP默认缓冲层
flush(); // 冲刷SAPI缓冲层
sleep(2);
```

注意ob_start()以后会创建一个用户缓冲区，它在默认缓冲层之上，而ob_end_flush()是冲刷缓冲层的最顶端，也就是刚刚创建的用户缓冲区，这时候输出从UC1刷到了默认缓冲层。

有人说用默认缓冲层就够了，还搞什么用户缓冲层。下面就来看看用户缓冲区一般用来干嘛。

### 用户缓冲层应用场景
1. 对第三方API输出做修改

在ThinkPHP框架中有个dump函数，是var_dump的改良版，能以更友好的格式显示变量结构：
```php
/**
 * 浏览器友好的变量输出
 * @access public
 * @param  mixed       $var   变量
 * @param  boolean     $echo  是否输出(默认为 true，为 false 则返回输出字符串)
 * @param  string|null $label 标签(默认为空)
 * @param  integer     $flags htmlspecialchars 的标志
 * @return null|string
 */
public static function dump($var, $echo = true, $label = null, $flags = ENT_SUBSTITUTE)
{
    $label = (null === $label) ? '' : rtrim($label) . ':';

    ob_start();
    var_dump($var);
    $output = preg_replace('/\]\=\>\n(\s+)/m', '] => ', ob_get_clean());

    if (IS_CLI) {
        $output = PHP_EOL . $label . $output . PHP_EOL;
    } else {
        if (!extension_loaded('xdebug')) {
            $output = htmlspecialchars($output, $flags);
        }

        $output = '<pre>' . $label . $output . '</pre>';
    }

    if ($echo) {
        echo($output);
        return;
    }

    return $output;
}
```
它里面就是用到了输出缓冲，将本来var_dump输出的内容放到一块用户缓冲区，然后再拿出来通过一个正则替换再输出，如果不借助输出缓冲的话做起来是很费力的，比如自己重新写一个。

### 总结
- ob_flush(): 冲刷PHP的默认输出缓冲层
- flush(): 冲刷SAPI缓冲区
- ob_start(): 创建用户缓冲区
- ob_get_clean(): 获取当前输出缓冲区的内容并删除当前输出缓冲区
- ob_end_flush(): 冲刷PHP的输出缓冲层（包括用户缓冲层和默认缓冲层）的最顶端的输出缓冲区
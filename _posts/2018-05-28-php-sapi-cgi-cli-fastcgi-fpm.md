---
layout: post
category: "php"
title:  "php sapi cgi cli fastcgi fpm"
---

## 什么是SAPI

SAPI: Server Application Programming Interface 服务端应用编程接口

![PHP架构](/images/php-arch.jpg)

PHP的SAPI就是应用层（apache、nginx、cli）和PHP进行数据交互的接口，常见的有Apache的mod_php5、CGI，IIS的ISAPI，还有Shell的CLI，PHP_SAPI常量保存着当前使用的是何种SAPI

```shell
C:\Users\think>php -r "echo PHP_SAPI;"
cli
```

```php
<?php
echo PHP_SAPI; // 输出apache2handler
```

## mod_php5
上面代码中apache2handler就是属于Apache的mod_php5模块的一部分，意味着PHP是作为Apache的子模块

就像我们经常用到的虚拟主机配置，它是作为Apache的一个子模块，当我们修改了vhost配置，只需要重启Apache，就可以读取到最新的配置。那么Apache是如何加载这些模块使之生效的呢？别急，先看看Apache的运行过程

### Apache的运行过程

Apache的运行分为启动阶段和运行阶段。 在启动阶段，Apache为了获得系统资源最大的使用权限，将以特权用户root（*nix系统）或超级管理员Administrator(Windows系统)完成启动， 并且整个过程处于一个单进程单线程的环境中。 这个阶段包括配置文件解析(如http.conf文件)、模块加载(如mod_php，mod_perl)和系统资源初始化（例如日志文件、共享内存段、数据库连接等）等工作。

Apache的启动阶段执行了大量的初始化操作，并且将许多比较慢或者花费比较高的操作都集中在这个阶段完成，以减少了后面处理请求服务的压力。

在运行阶段，Apache主要工作是处理用户的服务请求。 在这个阶段，Apache放弃特权用户级别，使用普通权限，这主要是基于安全性的考虑，防止由于代码的缺陷引起的安全漏洞。 Apache对HTTP的请求可以分为连接、处理和断开连接三个大的阶段。

### Apache的模块加载

Apache模块加载分为静态和动态

- 静态模块

	模块编译到httpd二进制文件中，Apache启动时加载，调用模块时直接运行
	
	优点：执行速度快

	缺点：修改模块需要重新编译Apache

- 动态模块

	模块编译成动态链接库，Apache运行时加载
	
	优点：修改模块不用重新编译Apache，只需要发送HUP信号重启Apache即可

	缺点：启动和执行速度有所降低

### PHP作为Apache动态模块的安装过程

详细内容可参考[PHP官方文档](http://php.net/manual/zh/install.unix.apache.php)

```shell
1.  gunzip apache_xxx.tar.gz
2.  tar -xvf apache_xxx.tar
3.  gunzip php-xxx.tar.gz
4.  tar -xvf php-xxx.tar
5.  cd apache_xxx
6.  ./configure --prefix=/www --enable-module=so
7.  make
8.  make install
9.  cd ../php-xxx
10. ./configure --with-mysql --with-apxs=/www/bin/apxs
11. make
12. make install

    如果在安装之后决定修改配置选项，那么只需重复以上最后三步。只须重新启动
    Apache 就可以使新模块生效。不需要重新编译 Apache。

13. cp php.ini-dist /usr/local/lib/php.ini


14. 编辑 httpd.conf 来加载 PHP 模块。在 LoadModule 语句右边的路径必须指向系统中
    PHP 模块所在的路径。上面的 make install 步骤可能已经添加了，但还是检查确认一下。

      LoadModule php5_module        libexec/libphp5.so

15. 在 httpd.conf 中加入 AddModule 部分，在 ClearModuleList 下面的某处，加上这一句：

      AddModule mod_php5.c

16. 告诉 Apache 将哪些后缀作为 PHP 解析。例如，让 Apache 把 .php 后缀的文件解析为
    PHP。可以将任何后缀的文件解析为 PHP，只要在以下语句中加入并用空格分开。这里以
    添加一个 .phtml 来示例。

      AddType application/x-httpd-php .php .phtml

    为了将 .phps 作为 PHP 的源文件进行语法高亮显示，还可以加上：

      AddType application/x-httpd-php-source .phps

17. 用通常的过程启动 Apache（必须完全停止 Apache 再重新启动，而不是用 HUP 或者
    USR1 信号使 Apache 重新加载）。
```

第10行的with-apxs是什么意思呢？

### apxs
apxs: Apache Extension Tool（注意不是asp.net文件的后缀）
> apxs是一个为Apache HTTP服务器编译和安装扩展模块的工具，用于编译一个或多个源程序或目标代码文件为动态共享对象，使之可以用由mod_so提供的**LoadModule**指令在运行时加载到Apache服务器中。
> 
> Apache动态编译依赖于mod_so模块，它是静态编译到httpd二进制文件里的，随着Apache一起启动

也就是说apxs是Apache用来安装动态模块的工具，在编译PHP的时候，加上--with-apxs就是把PHP作为Apache的动态模块

### mod_php5是如何让Apache和PHP协同工作的？

1. 启动Apache，解析配置文件，加载模块，mod_php5.so被加载到内存中，启动多个httpd进程监听客户端请求
1. 启动完成，等待客户端请求
1. 客户端请求index.php
1. 其中一个httpd进程调用mod_php5模块
1. mod_php5模块调用PHP解析器、引擎，解析和执行PHP代码
1. mod_php5给httpd返回执行结果
1. httpd给客户端返回响应结果

### mod_php5的优缺点
- 优点：每个Apache进程启动的时候，PHP的环境已经初始化好，请求来的时候，直接就可以解释执行目标PHP文件。

- 缺点：与Apache耦合度比较深，如果使用了opcode缓存的话，PHP代码更改了，要重启Apache

## CGI
CGI: Common Gateway Interface 通用网关接口

Web Server每收到一个请求，就会启动一个CGI进程来处理，CGI处理返回结果，然后结束进程
和mod_php5模块方式相比好处是减少了Web Server和PHP的耦合，但是每次请求都会重新加载php配置和执行一些初始化工作，进程不停地启动和结束会造成很大开销

## FastCGI

FastCGI是Web服务器和处理程序之间通信的一种协议， 是CGI的一种改进方案，FastCGI像是一个常驻(long-lived)型的CGI， 它可以一直执行，在请求到达时不会花费时间去fork一个进程来处理(这是CGI最为人诟病的fork-and-execute模式)。 正是因为他只是一个通信协议，它还支持分布式的运算，所以 FastCGI 程序可以在网站服务器以外的主机上执行，并且可以接受来自其它网站服务器的请求。

FastCGI 是与语言无关的、可伸缩架构的 CGI 开放扩展，将 CGI 解释器进程保持在内存中，以此获得较高的性能。 CGI 程序反复加载是 CGI 性能低下的主要原因，如果 CGI 程序保持在内存中并接受 FastCGI 进程管理器调度， 则可以提供良好的性能、伸缩性、Fail-Over 特性等。

### FastCGI 工作流程

FastCGI 进程管理器自身初始化，启动多个 CGI 解释器进程，并等待来自 Web Server 的连接。
Web 服务器与 FastCGI 进程管理器进行 Socket 通信，通过 FastCGI 协议发送 CGI 环境变量和标准输入数据给 CGI 解释器进程。
CGI 解释器进程完成处理后将标准输出和错误信息从同一连接返回 Web Server。
CGI 解释器进程接着等待并处理来自 Web Server 的下一个连接。

### PHP-FPM

PHP-FPM 是对于 FastCGI 协议的具体实现，他负责管理一个进程池，来处理来自Web服务器的请求

## 参考文档
- [php手册-安装-Unix 系统下的 Apache 1.3.x](http://php.net/manual/zh/install.unix.apache.php)
- [Apache DSO](http://httpd.apache.org/docs/2.4/zh-cn/dso.html)
- [深入理解Zend SAPIs](http://www.laruence.com/2008/08/12/180.html)
- [深入理解PHP内核](http://www.php-internals.com/book/?p=chapt02/02-02-00-overview)
- [C++静态库与动态库](http://www.cnblogs.com/skynet/p/3372855.html)
- [CGI、FastCGI和PHP-FPM关系图解](https://www.awaimai.com/371.html)
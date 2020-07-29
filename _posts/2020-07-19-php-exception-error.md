---
layout: post
category: "php"
title:  "PHP的异常和错误"
---

PHP5一般直接抛出错误，而非异常，大部分情况下，只有你自己抛出的异常才能被捕获，除了一些内建的异常，比如：PDOException, ReflectionException, [SPL Exceptions](https://www.php.net/manual/en/spl.exceptions.php)。

PHP7增加了Throwable接口，PHP 7 改变了大多数错误的报告方式。不同于传统（PHP 5）的错误报告机制，现在大多数错误被作为 Error 异常抛出。



---
layout: post
title:  php安装扩展的几种方法
categories: php
---

### apt或yum

*Debian或者Ubuntu:*

1. apt-cache search memcached php
2. apt-get install -y php5-memcached

*CentOS:*

1. yum search memcached php
2. yum install -y php-pecl-memcached

### pecl

1. pecl install memcached
2. php --ini查看php.ini文件位置，然后在文件中添加extension=memcached.so

### phpize

1. /path/to/sourcedir
1. phpize
2. ./configure --with-php-config=/path/to/php-config
3. make && make install
4. php --ini查看php.ini文件位置，然后在文件中添加extension=memcached.so
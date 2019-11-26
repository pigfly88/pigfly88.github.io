---
layout: post
category: "mix"
title:  "使用ELK搭建日志平台"
---

## Elasticsearch

Elasticsearch 是一个分布式、可扩展、实时的搜索与数据分析引擎。经常用来做全文搜索、结构化数据的实时统计。

索引相当于关系数据库中的数据库 ，类型相当于是关系数据库的表。

关系型数据库使用B数索引，Elasticsearch使用倒排索引。

LogStash 是开源的服务器端数据处理管道，能够同时从多个来源采集数据，转换数据，然后将数据发送到Elasticsearch中。

Beats 是轻量级的采集器，采用Go编写，相比于使用JVM的Elasticsearch，开销更小，适合于在各个服务器上搜集日志后传输给Logstash，包含以下类型：

Filebeat 是 Beats 系列中的一种，Filebeat 支持从文件中收集数据，还支持许多其他数据源，包括 TCP/UDP、容器、Redis 和 Syslog。借助丰富的模块，可以轻松针对 Apache、MySQL 和 Kafka 等常见应用程序的日志格式收集和解析相应的数据。

Kibana 对Elasticsearch的数据做可视化。

安装步骤按照官方文档来就可以了：https://www.elastic.co/cn/products/log-monitoring

装好以后需要注意调整一些配置：

设置Kibana的host，默认是localhost，这样的话不能从其他机器进行访问，我这里是设置为开发服的IP：

server.host: "10.250.160.174"

设置Filebeat需要采集的日志目录，默认采集/var/log目录，因为我们要采集项目文件夹下面的日志，加上一行就行了：

- type: log
  enabled: true
  paths:
    - /workspace/data/www/data-warehouse/var/log/*.log
    - /var/log/*.log
	
默认是一行被认为是一条日志内容，我们需要把下面这种日志内容解析为一条消息：

[17-Sep-2019 11:39:57 Asia/Shanghai] PHP Fatal error:  Uncaught Symfony\Component\Debug\Exception\FatalThrowableError: syntax error, unexpected '$this' (T_VARIABLE) in /workspace/data/www/data-warehouse/src/Manager/Kom/KomMaterialModelManager.php:305
Stack trace:
#0 [internal function]: Symfony\Component\Debug\DebugClassLoader->loadClass('App\\Manager\\Kom...')
#1 [internal function]: spl_autoload_call('App\\Manager\\Kom...')
#2 /workspace/data/www/data-warehouse/vendor/symfony/config/Resource/ClassExistenceResource.php(78): class_exists('App\\Manager\\Kom...')
#3 /workspace/data/www/data-warehouse/vendor/symfony/dependency-injection/ContainerBuilder.php(353): Symfony\Component\Config\Resource\ClassExistenceResource->isFresh(0)
#4 /workspace/data/www/data-warehouse/vendor/symfony/dependency-injection/Loader/FileLoader.php(150): Symfony\Component\DependencyInjection\ContainerBuilder->getReflectionClass('App\\Manager\\Kom...')
#5 /workspace/data/www/data-warehouse/vendor/symfony/dependency-injection/Loader/FileLoader.php(57): Symfony\Component\DependencyInjection\L in /workspace/data/www/data-warehouse/src/Manager/Kom/KomMaterialModelManager.php on line 305

设置多行解析规则：

### Multiline options
multiline.pattern: ^\[
multiline.negate: true
multiline.match: after

- type: log
  enabled: true
  paths:
    - /workspace/data/www/data-warehouse/var/log/*.log
  fields:
    pdw: true

  tags: ["dw"]
 
  ### Multiline options
  multiline.pattern: ^\[
  multiline.negate: true
  multiline.match: after
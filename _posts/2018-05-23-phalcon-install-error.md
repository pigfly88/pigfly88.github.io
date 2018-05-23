---
layout: post
category: "php"
title:  "安装Phalcon报错：gcc: Internal error: Killed (program cc1)"
---

## 起因

安装Phalcon可以参考github上面的[README.md](https://github.com/phalcon/cphalcon)

下面是我在阿里云ECS服务器上面执行命令的过程：

```shell
# 安装依赖
sudo yum install php-devel pcre-devel gcc make re2c

# 编译安装
git clone git://github.com/phalcon/cphalcon.git
cd cphalcon/build

# 这里最好指定一下php的具体路径，以免有多个php版本的时候安装到别的版本里面去了
# 可以用php打印phpinfo()信息查看当前php版本和路径信息
sudo ./install --phpize /alidata/server/php/bin/phpize --php-config /alidata/server/php/bin/php-config
```

然后发现报错如下：

```shell
gcc: Internal error: Killed (program cc1)
```

bing搜索一下，发现有人遇到过类似问题，原因是阿里云ECS内存不足并且默认关闭了swap引起的。

于是copy and execute，问题解决：

```shell
#创建交换分区目录
sudo mkdir -p /var/cache/swap/

#创建用于交换分区的文件。count=512 代表设置512MB大小swap文件
sudo dd if=/dev/zero of=/var/cache/swap/swap0 bs=1M count=512
sudo chmod 0600 /var/cache/swap/swap0

#设置交换分区文件
sudo mkswap /var/cache/swap/swap0 

#立即启用交换分区文件
sudo swapon /var/cache/swap/swap0
```

## 后续

阿里云服务器初始状态未配置swap，是因为开启swap分区会导致硬盘IO性能下降。执行如下命令关闭swap：

```shell
# 关闭swap
swapoff /var/cache/swap/swap0

# 查看swap状态
swapon -s
```
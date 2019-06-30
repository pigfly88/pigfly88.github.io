---
layout: post
category: "php"
title:  "用Docker搭建LNMP"
---

程序员经常会说的一句话：在我的机器上是正常的，肯定是你的机器有问题。因此，Docker诞生了，它把应用所需要的一切东西都打包，从而可以很方便地进行部署。

Docker 的主要用途，目前有三大类：

1. 提供一次性的环境。比如，本地测试他人的软件、持续集成的时候提供单元测试和构建的环境。
1. 提供弹性的云服务。因为 Docker 容器可以随开随关，很适合动态扩容和缩容。
1. 组建微服务架构。通过多个容器，一台机器可以跑多个服务，因此在本机就可以模拟出微服务架构。

下面开始来体验Docker，操作系统为win10，Docker的安装请参考[官方文档](https://docs.docker.com/docker-for-windows/install/)，这里略过。

### Hello World

官方提供了一个程序员最喜欢的hello world镜像，我们就以此来开始Docker之旅，感受Docker的强大之处吧：

```bash
docker run hello-world
```

如果你看到输出```Hello from Docker!```，恭喜你，Docker安装成功了。

### image和container

镜像和容器可以理解为面向对象里面的类和对象。

上面的例子是直接拿官方的镜像来跑，我们也可以在官方镜像的基础上加工，来构建新的镜像，当然也可以不基于任何镜像来构建。

所有内容（代码、软件、环境等）都可以通过Dockerfile配置来构建image，通过image可以运行具体的container实例。

查看镜像列表：```docker image ls```

查看容器列表：```docker container ls```

如果要查看完整的列表，在```ls```后面加个```-a```选项就好了：```docker container ls -a```

### 装个Linux
下面加点难度，装个Linux，alpine体积小，很多开源镜像也是以它为基础（比如Nginx），很适合用来体验：

```bash
docker run -it --rm alpine
```

发现run后面多了几个选项，其中```-it```其实是```-i -t```的缩写，即interactive terminal，交互式终端的意思，不加这个我们是进不到容器的交互式终端的。

加了--rm，表示在容器退出后会自动删除容器，我们这里是为了体验，希望用完之后不要占用磁盘空间，所以加了删除选项，注意这里不是删除镜像，删除的是容器。

现在我们进入到alpine系统的终端了，输入命令查看系统版本试试：

```bash
cat /etc/os-release
```

体验完毕，这时候只要输入```exit```按下回车就可以退出容器了。

如果想运行容器的时候直接输出系统版本，可以这么玩：

```
docker run --rm alpine cat /etc/os-release
```

在镜像名后面直接跟上你想运行的命令，输出系统版本信息以后容器就自动关闭并删除了。

### 运行PHP

好了，现在Linux系统装好了，直觉告诉我，如果要跑PHP，我们就直接进到刚刚那个alpine容器里面安装就好了，那么不好意思，你的姿势错了。先不解释，先看看正确姿势：

我们先在CLI模式下体验PHP，在这之前，我们可以看看官方的php-cli:7.2镜像的[Dockerfile](https://github.com/docker-library/php/blob/a9f19e9df5f7a5b74d72a97439ca5b77b87faa35/7.2/stretch/cli/Dockerfile)是怎么定义的：
```
FROM debian:stretch-slim
...此处省略若干行...
```

FROM指定基于哪个镜像，也可以不基于任何镜像：```FROM scratch```，比如官方的alpine的[Dockerfile](https://github.com/alpinelinux/docker-alpine/blob/c13cf91002136261e690adacd43a8fd6b929b4c8/x86_64/Dockerfile)：

```
FROM scratch
ADD alpine-minirootfs-3.10.0-x86_64.tar.gz /
CMD ["/bin/sh"]
```

由此可见，PHP镜像是建立在debian这个镜像的基础上的，所以我们不需要像在虚拟机上面装PHP一样，得先用iso安装一个Linux操作系统。

项目位置在D:\www\php-cli-demo，建立好文件夹以后，在项目下面新建一个```index.php```文件：
```php
<?php
echo phpversion();
```

再新建一个```Dockerfile```文件：
```
FROM php:7.2-cli
WORKDIR /var/www/php-cli-demo
COPY . .
CMD ["php", "index.php"]
```

WORKDIR是指定容器的工作目录，我们需要把当前项目的代码拷贝到容器的操作系统所在的目录里，所以我们使用COPY，第一个点的意思是当前目录，第二个点的意思是容器的当前目录，意思就是把我win10系统下D:\www\php-cli-demo这个文件夹下的所有文件全部拷贝到容器的Debian系统下的当前目录（已通过工作目录指定为/var/www/php-cli-demo）下。

CMD就是容器运行起来以后所要执行的命令了，这里我们希望直接运行这个php文件。

在当前文件夹下，按住```Shift```键，然后按下鼠标右键，出现右键菜单，这时候出现其中一项“在此处打开Powershell窗口（s）”，这时候我们只需要按下```s```就可以进入命令行并且直接定位到当前文件夹了。

然后我们根据刚才的Dockerfile配置文件来建立镜像：

```
docker build -t php-cli-demo .
```

t是tag，打标签的意思。

最后那个点，指定了要打包的是哪个目录，docker build 命令得知这个路径后，会将路径下的所有内容打包，然后上传给 Docker 引擎。这样 Docker 引擎收到这个上下文包后，展开就会获得构建镜像所需的一切文件，然后Docker引擎在解析上面的COPY指令时，就能知道要COPY哪些文件了，所以这个COPY的第一个参数的路径是在打包路径下展开的，是相对路径。

Docker默认会去找Dockerfile，也可以是别的文件名，例如你的Dockerfile叫build.conf，可以通过```-f```参数指定：
```
docker build -t php-cli-demo -f build.conf .
```

好了，镜像构建成功了，现在马上来跑一下吧：
```
docker run --rm php-cli-demo
```

可以看到正确输出了PHP的版本号为：7.2.19。

build的时候把项目代码一块打包进image镜像文件了，那如果我要运行另外一个项目呢？要重新build一遍吗？其实这个image是可以复用的，只不过在运行的时候指定项目位置就行了：

```
docker run --rm -v ${PWD}:/var/www/php-cli-demo php-cli-demo
```

v是volume，数据卷的意思，${PWD}是宿主机当前目录，冒号后面是容器目录，意思就是把当前目录挂载到容器的/var/www/php-cli-demo目录。

不过不建议这么做，因为镜像是一层一层做缓存的，所以重复build不会占用太多空间，而且如果镜像做了改动，会影响到另外一个容器，所以还是单独build比较好。感兴趣的朋友可以自己做一下测试，然后用```docker system df```查看空间占用情况。

如果像上面这种在基础镜像上，没有涉及到任何加工的，只需要把项目映射到容器就行了，可以直接这么玩：

```bash
docker run --rm -v ${PWD}:/var/www/demo -w /var/www/demo php:7.2-cli php index.php
```

现在，回过头来分析刚才的问题，为什么不建议在alpine容器里面直接安装PHP。

镜像构建时，会一层一层地构建，前一层是后一层的基础，这样可以达到复用的效果。

容器的实质是进程，但与直接在宿主执行的进程不同，容器进程运行于属于自己的独立的命名空间。因此容器可以拥有自己的 root 文件系统、自己的网络配置、自己的进程空间，甚至自己的用户 ID 空间。容器内的进程是运行在一个隔离的环境里，使用起来，就好像是在一个独立于宿主的系统下操作一样。这种特性使得容器封装的应用比直接在宿主运行更加安全。

镜像使用的是分层存储，容器也是如此。每一个容器运行时，是以镜像为基础层，在其上创建一个当前容器的存储层。

容器存储层的生存周期和容器一样，容器消亡时，容器存储层也随之消亡。因此，任何保存于容器存储层的信息都会随容器删除而丢失。

所以，你在alpine容器里面直接安装PHP，如果不小心把容器删掉了，下一次再跑的时候，PHP就丢了。而且这是一个黑箱操作，别人并不知道这个容器里面有什么，所以最好还是通过Dockerfile构建镜像，然后再根据镜像来跑容器咯。

### 搭建LNMP
好了，上面的都搞明白之后我们就来跑个真正的PHP项目吧。我们需要用到PHP-FPM，Nginx，MySQL。项目结构是这么安排的：

- docker-php-demo
	- app
		- index.html
		- index.php
	- mysql
		- Dockerfile
	- nginx
		- conf.d
			- site.conf
	- php
		- Dockerfile

我们先把Nginx装起来，跑个静态页面试试。直接跑官方镜像：
```
docker run --name nginx -d --rm nginx:1.16
```

打开浏览器输入localhost，就可以看到Welcome to nginx!的静态页了。这是Nginx的默认页面，我们需要加一个专门针对本次项目的配置。进入Nginx容器，查看配置文件路径：

```
docker exec -it nginx bash
nginx -V
```

看到配置文件路径为： 
```
--conf-path=/etc/nginx/nginx.conf
```

查看配置文件:
```
cat /etc/nginx/nginx.conf
```

可以看到默认的配置还会去加载其他配置文件：
```
include /etc/nginx/conf.d/*.conf;
```

所以我们只需要把项目的配置通过```-v```映射到容器的conf.d目录下就可以了。把当前的Nginx容器停掉：
```
docker stop nginx
```

然后编写site.conf：

```
server {
    listen 80;
    server_name local.docker-php-demo.com;
    root /var/www/docker-php-demo;
}
```

在app目录放入index.html：
```
Hello Docker
```

因为Nginx本身我们不做加工，所以不用Dockerfile，直接用官方镜像就好了，然后分别把项目和Nginx网站配置都映射过去，在项目根目录下执行：

```bash
docker run --name dpd-nginx -d -v ${PWD}/app:/var/www/docker-php-demo -v ${PWD}/nginx/conf.d:/etc/nginx/conf.d -p 80:80 nginx:1.16
```

最后在本机配置hosts就大功告成啦，Win + R 打开运行窗口，输入drivers，打开etc/hosts文件加入一行域名配置：

```
127.0.0.1 local.docker-php-demo.com
```

浏览器打开local.docker-php-demo.com，到这里，我们就完成了让Nginx跑静态页面的配置了。我们的项目是PHP，把fpm装起来，然后再让Nginx连上就行了，在php目录下编写Dockerfile：

```
FROM php:7.2-fpm
RUN cp /usr/local/etc/php/php.ini-development /usr/local/etc/php/php.ini \
    && docker-php-ext-install pdo_mysql
```

上面我们使用了镜像提供的安装扩展的快捷操作方法来安装pdo_mysql扩展。

在app目录放入index.php：
```
<?php
phpinfo();
```

构建运行：

```bash
cd php
docker build -t dpd-php .
cd ..
docker run --name dpd-php -d -v ${PWD}/app:/var/www/docker-php-demo -p 9000:9000 dpd-php
```

把php和nginx通过网卡连接起来：
```
docker network create --driver bridge dpd # 新建一个桥接网卡，名字叫dpd
docker network connect dpd dpd-php
docker network connect dpd dpd-nginx
docker network ls # 列出docker当前有哪些网卡
docker network inspect dpd # 查看dpd网卡详情
```

修改Nginx配置，把PHP脚本转发给fpm，fpm地址通过容器名dpd-php就可以识别：

```
server {
    listen 80;
    server_name local.docker-php-demo.com;
    root /var/www/docker-php-demo;
    
    location ~ \.php$ {
        fastcgi_pass dpd-php:9000;
        include fastcgi_params;
        fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}
```

让Nginx重新加载配置文件：

```
docker exec -it dpd-nginx nginx -s reload
```

到这里，我们就把Nginx 和 php-fpm搭起来了。下面我们再来搭建Mysql，在mysql目录下新建test.sql，存放表结构和基础数据：

```
CREATE TABLE `user` (
    `id` int(10) unsigned NOT NULL PRIMARY KEY AUTO_INCREMENT,
    `name` varchar(20) NOT NULL
)ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='用户';

INSERT INTO `user` VALUES (null, 'a1');
```

Dockerfile：

```
FROM mysql:5.7
ENV MYSQL_ROOT_PASSWORD=123 MYSQL_DATABASE=test
COPY ./test.sql /var/data/test.sql
```

构建并启动容器，这里直接用```--network```参数把容器加入dpd网络：
```
cd mysql
docker build -t dpd-mysql .
docker run --name dpd-mysql -d --network dpd -p 3306:3306 dpd-mysql
```

进入容器把表和测试数据导入进去：
```
docker exec -it dpd-mysql bash
mysql -uroot -p
mysql> use test;
mysql> source /var/data/test.sql
```

修改index.php：
```php
<?php
$dsn = 'mysql:dbname=test;host=dpd-mysql';
$user = 'root';
$password = '123';

try {
    $dbh = new PDO($dsn, $user, $password);
    $sql = 'SELECT * FROM user WHERE id=?';
    $sth = $dbh->prepare($sql);
    $sth->execute(array(1));
    $result = $sth->fetch(PDO::FETCH_ASSOC);
    var_dump($result);
} catch (PDOException $e) {
    echo 'Error: ' . $e->getMessage();
}
```

浏览器输入[http://local.docker-php-demo.com/index.php](http://local.docker-php-demo.com/index.php)，PHP成功地从数据库获取到了用户数据：
```php
array(2) { ["id"]=> string(1) "1" ["name"]=> string(2) "a1" }
```

### 参考资料
- [Docker 官方文档](https://docs.docker.com/get-started/)
- [Docker 入门教程](http://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html)
- [Docker — 从入门到实践](https://docker_practice.gitee.io/)
- [Docker for beginners](https://github.com/docker/labs/tree/master/beginner)
- [Setting up PHP, PHP-FPM and NGINX for local development on Docker](https://www.pascallandau.com/blog/php-php-fpm-and-nginx-on-docker-in-windows-10/)
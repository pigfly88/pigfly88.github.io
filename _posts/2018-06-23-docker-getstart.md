---
layout: post
category: "linux"
title:  "用Docker构建PHP项目"
---

Docker就是把应用所需要的一切东西都打包，从而可以很方便地进行部署。

## 安装Docker

```bash
$ sudo yum install -y yum-utils device-mapper-persistent-data lvm2
$ sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
$ sudo yum install docker-ce
$ sudo systemctl start docker
```

把当前用户加入Docker用户组

```bash
$ sudo groupadd docker
$ sudo usermod -aG docker $USER # $USER是你的用户名
```

### 体验

```bash
$ docker image pull library/hello-world
$ docker run hello-world
```

### image和container

所有内容（代码、依赖、环境等）会打包成一个image文件，container是image的实例。

## 打包

```bash
# 假设现在是刚开始编写项目，创建项目文件夹，如果要在已有项目上打包，跳到下一步
$ cd ~
$ mkdir docker-demo
$ cd docker-demo # 进入项目目录
$ vi index.php

# 输入下面内容，:wq保存退出
<?php
echo phpversion();
```

```bash
$ vi Dockerfile # 在项目目录里面新建docker配置

# 输入下面内容，:wq保存退出
FROM php:5.6-zts-alpine
COPY . /usr/src/myapp
WORKDIR /usr/src/myapp
CMD [ "php", "./index.php" ]
```

```bash
$ docker build -t docker-demo .
$ docker run --rm docker-demo
5.6.36$
```
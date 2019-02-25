---
layout: post
category: "linux"
title:  "网站根目录的权限设置"
---

```shell
# 查看apache的配置文件
$ vi /alidata/server/httpd/conf/httpd.conf

# 搜索group关键字
$ /group

# 看到用户和用户组都是www
User www
Group www

# 把网站根目录的所属用户和用户组设置为对应的user和group
$ chown -R www:www /alidata/www/project
```
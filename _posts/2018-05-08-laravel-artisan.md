---
layout: post
category: "php"
title:  "Laravel artisan常用命令"
---


```shell
composer create-project --prefer-dist laravel/laravel myapp # 创建项目

php artisan list # 命令列表

php artisan help seed # 当你忘记命令了，这样会有提示

php artisan make:model Article -m # 创建模型并且生成数据迁移文件

php artisan migrate # 执行数据迁移

php artisan make:seeder ArticleSeeder # 创建数据填充文件

php artisan db:seed --class=ArticleSeeder # 执行数据填充

php artisan make:controller ArticleController --resource # 创建控制器 并且是个资源控制器（含有默认restful方法）
```
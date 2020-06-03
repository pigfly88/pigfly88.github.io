---
layout: post
title:  linux常用命令
categories: linux
---

### 文件和目录
- cat 1.log | grep passwd

	查看1.log内容，并匹配passwd相关的内容
- tail -n 20 -f 1.log

	显示1.log最后20行内容，f为follow的意思，动态跟踪，如果文件内容有更新会及时更新
- find / -name 1.log
- chmod -R 755 /usr/www
	
	r:4, w:2, x:1
- chown -R www:www /usr/www
- df -h

	查看磁盘剩余空间。df: disk free，h为human的意思，即人性化显示

```
查看文件里面匹配内容的前后几行
cat /workspace/log/test.log | grep -50 'not connect to'
```

磁盘空间满了，怎样快速查看是哪个目录或文件占用空间大:
du -m --max-depth=1 /workspace/log/ | sort -nr
du -ah --max-depth=1 /
du -ah --max-depth=1 /workspace/data | grep 'G' | sort -nr

### 用户权限
- useradd
- userdel

### 网络相关
- ss -lp | grep 3306
	查看3306端口状态。ss: socket status，查看socket状态。

添加PATH变量

vi /etc/profile
export PATH=/usr/local/bin:$PATH
soruce /etc/profile
---
layout: post
category: "linux"
title:  "添加环境变量"
---

```shell
export PATH=$PATH:/data/services/php7/bin/
```

上面的方法只针对当前SESSION有效，如果需要设置全局有效，可以修改配置文件：
```shell
vi /etc/profile
# 在最后面加入:
export PATH="$PATH:/opt/au1200_rm/build_tools/bin"
# 让环境变量立即生效：
source /etc/profile
```

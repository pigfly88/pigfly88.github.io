---
layout: post
category: "linux"
title:  "svn命令"
---

```shell
# 更新
svn up

# 提交
svn ci -m "commit"

# 撤销更改（还没提交）
svn revert b/public/index.php

# 回滚到某个版本
svn merge -r 457:456 common/config/config.ini

# 忽略提交
svn pset svn:ignore index.php ./mp/public/

# 忽略更新文件
svn update --set-depth files common/config/config.ini

# 忽略更新文件夹
svn update --set-depth exclude common/config
```
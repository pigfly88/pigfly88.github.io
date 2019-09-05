---
layout: post
category: "mysql"
title:  "MySQL的char和varchar"
---

- 最大长度：char是255，varchar是65535
- char在存储时，如果字符长度不足，会自动填补空格到指定长度，查询时返回的结果会自动删除尾部的空格。
- 跟char相比，varchar需要额外的1~2个字节来存储字符长度。
> [The CHAR and VARCHAR Types](https://dev.mysql.com/doc/refman/5.7/en/char.html)
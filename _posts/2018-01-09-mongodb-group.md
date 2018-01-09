---
layout: post
title:  MongoDB分组查询
categories: nosql
---

### 插入测试数据
```shell
$ mongod #启动MongoDB
$ mongo #连接MongoDB

> m1={
... "id":1,
... "title":"危情24小时"
... };
> m2={
... "id":2,
... "title":"女大当嫁"
... };

> db.movies.insert(m1);
> db.movies.insert(m2);

> r1={
... "uid":1,
... "mid":1,
... "rating":3,
... "ctime":new Date()
... }
> r2={
... "uid":1,
... "mid":2,
... "rating":5,
... "ctime":new Date()
... }
> r3={
... "uid":2,
... "mid":1,
... "rating":4,
... "ctime":new Date()
... }
> r4={
... "uid":3,
... "mid":1,
... "rating":4,
... "ctime":new Date()
... }

> db.ratings.insert(r1);
> db.ratings.insert(r2);
> db.ratings.insert(r3);
> db.ratings.insert(r4);

> u1={
... "uid":1,
... "sex":0,
... 'username':'pigfly',
... }
> u2={
... "uid":2,
... "sex":1,
... 'username':'zch',
... }
> u3={
... "uid":3,
... "sex":0,
... 'username':'justlikeheaven',
... }

> db.users.insert(u1);
> db.users.insert(u2);
> db.users.insert(u3);
```

测试数据插入完毕，看起来像是下面这样：
```shell
> db.movies.find();
{ "_id" : ObjectId("5a543db21852eaeaf54508cd"), "id" : 1, "title" : "危情24小时" }
{ "_id" : ObjectId("5a543db51852eaeaf54508ce"), "id" : 2, "title" : "女大当嫁" }

> db.ratings.find();
{ "_id" : ObjectId("5a543ec11852eaeaf54508d2"), "uid" : 1, "mid" : 1, "rating" : 3, "ctime" : ISODate("2018-01-09T03:59:08.523Z") }
{ "_id" : ObjectId("5a543ec31852eaeaf54508d3"), "uid" : 3, "mid" : 1, "rating" : 4, "ctime" : ISODate("2018-01-09T03:59:51.968Z") }
{ "_id" : ObjectId("5a543eea1852eaeaf54508d4"), "uid" : 2, "mid" : 1, "rating" : 1, "ctime" : ISODate("2018-01-09T04:02:42.159Z") }
{ "_id" : ObjectId("5a543f0a1852eaeaf54508d5"), "uid" : 1, "mid" : 2, "rating" : 5, "ctime" : ISODate("2018-01-09T04:03:19.448Z") }

> db.users.find();
{ "_id" : ObjectId("5a543e9b1852eaeaf54508cf"), "uid" : 2, "sex" : 1, "username" : "zch" }
{ "_id" : ObjectId("5a543e9d1852eaeaf54508d0"), "uid" : 1, "sex" : 0, "username" : "pigfly" }
{ "_id" : ObjectId("5a543ea01852eaeaf54508d1"), "uid" : 3, "sex" : 0, "username" : "justlikeheaven" }

```

### 使用group分组查询
查询所有电影的总评分：
```shell
db.runCommand(
   {
     group:
       {
         ns: 'ratings',
         key: { mid:true },
         cond: { ctime: { $gt: new Date( '01/01/2018' ) } },
         $reduce: function ( curr, result ) { result.total_ratings += curr.rating },
         initial: { total_ratings:0 }
       }
   }
)
```

返回：
```json
{
        "retval" : [
                {
                        "mid" : 1,
                        "total_ratings" : 8
                },
                {
                        "mid" : 2,
                        "total_ratings" : 5
                }
        ],
        "count" : NumberLong(4),
        "keys" : NumberLong(2),
        "ok" : 1
}
```
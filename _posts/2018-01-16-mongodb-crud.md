---
layout: post
title:  MongoDB CRUD
categories: nosql
---

```shell
$ mongod #启动MongoDB
$ mongo #连接MongoDB

> t1={
... 'title':'xxx',
... }
{ "title" : "xxx" }
> db.test.insert(t1);
WriteResult({ "nInserted" : 1 })
> db.test.insert({'_id':1, 'age':21});
WriteResult({ "nInserted" : 1 })
> db.test.insertOne({'_id':2, 'age':31});
{ "acknowledged" : true, "insertedId" : 2 }
> db.test.insertOne({'age':51});
{
        "acknowledged" : true,
        "insertedId" : ObjectId("5a5d4ede5cf711278bafc55a")
}
> db.test.insertMany([{'sex':0},{'sex':1}])
{
        "acknowledged" : true,
        "insertedIds" : [
                ObjectId("5a5d4f235cf711278bafc55b"),
                ObjectId("5a5d4f235cf711278bafc55c")
        ]
}
> db.test.insert([{'title':'yy'}, {'title':'zz'}])
BulkWriteResult({
        "writeErrors" : [ ],
        "writeConcernErrors" : [ ],
        "nInserted" : 2,
        "nUpserted" : 0,
        "nMatched" : 0,
        "nModified" : 0,
        "nRemoved" : 0,
        "upserted" : [ ]
})
> db.test.find({$or:[{sex:0}, {sex:1}]})
> db.test.find()
{ "_id" : ObjectId("5a5d4de75cf711278bafc559"), "title" : "xxx" }
{ "_id" : 1, "age" : 21 }
{ "_id" : 2, "age" : 31 }
{ "_id" : ObjectId("5a5d4ede5cf711278bafc55a"), "age" : 51 }
{ "_id" : ObjectId("5a5d4f235cf711278bafc55b"), "sex" : 0 }
{ "_id" : ObjectId("5a5d4f235cf711278bafc55c"), "sex" : 1 }
{ "_id" : ObjectId("5a5d4f7d5cf711278bafc55d"), "title" : "yy" }
{ "_id" : ObjectId("5a5d4f7d5cf711278bafc55e"), "title" : "zz" }
> db.test.update({'age':{$gt:30}},{$set:{status:0}},{multi:true})
WriteResult({ "nMatched" : 2, "nUpserted" : 0, "nModified" : 2 })
> db.test.remove({'title':'yy'});
WriteResult({ "nRemoved" : 1 })
> db.test.deleteOne({'title':'zz'});
{ "acknowledged" : true, "deletedCount" : 1 }
> db.test.remove({status:0});
WriteResult({ "nRemoved" : 2 })
```
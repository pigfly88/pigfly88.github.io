---
layout: post
title:  javascript数组
categories: javascript
---

### 定义数组

```javascript
var a = [1, 2, 3];
typeof a; //object, 数组是对象
a.length; //数组长度
```

### 相关操作

```javascript
a[0]; //下标访问
a.push(4); //插入
a.pop(); //删除最后一个元素
a.join(''); //拼接成字符串

//遍历
var str = "";
for(var i in a){
    str += a[i]+' ';
}
str; //1234
```

### 关联数组

```javascript
var a = {'name':'pigfly', 'age':31};
a.name; //pigfly
a['age']; //31
delete a.age; //删除元素

//遍历
var str = "";
for(var i in a){
    str+=i+':'+a[i]+' ';
}
```
---
layout: post
title:  javascript数组
categories: javascript
---

### 定义数组

```javascript
var a = [1, 2, 3];
typeof a; //"object", 数组是对象
a.length; //数组长度
```

### 相关操作

```javascript
a[0]; //下标访问
a.push(4); //在数组末尾添加元素
a.pop(); //删除最后一个元素
a.join(''); //拼接成字符串

//遍历
var str = "";
for(var i in a){
    str += a[i]+' ';
}
```

### 关联数组

```javascript
var a = {'name':'pigfly', 'age':31};
a.name; //属性访问，也可以写成a['name']
delete a.age; //删除元素

//遍历
var str = "";
for(var i in a){
    str+=i+':'+a[i]+' ';
}
```

### 多维数组

```javascript
var a=[{'name':'zz'}, {'name':'dd', 'cid':[1,2,3]}];
```

### php数组转js数组

```php
$users = array(array('name'=>'zhupp'),array('name'=>'zz', 'address'=>array('province'=>'gd', 'city'=>'sz')));
echo 'var a = '.json_encode($users); //var a = [{"name":"zhupp"},{"name":"zz","address":{"province":"gd","city":"sz"}}]
```
---
layout: post
title:  JavaScript一看就懂(5)面向对象
categories: javascript
---

### 创建对象

#### 通过对象字面量创建对象

```javascript
var Book = {
    name: 'js',
    desc: function(){
        alert('my name is '+this.name);
    }
};
Book.name; //js
Book.desc(); //my name is js
```

#### 通过构造函数创建对象

```javascript
function Book(name){
    this.name = name;
    
    this.desc = function(){
        alert('my name is '+this.name);
    }
}

var b = new Book('javascript');
b.name; //javascript
b.desc(); //my name is javascript
```

也可以把方法写到原型上，这样函数只创建一次，并且所有实例都可以重用：

```javascript
function Book(name){
    this.name = name;
}

Book.prototype.desc = function(){
    alert('my name is '+this.name);
};

var b = new Book('php');
b.name; //php
b.desc(); //my name is php
```

### 继承

#### 通过原型继承
```javascript
function Book(name){
    this.name = name || '';
}
Book.prototype.desc = function(){
    alert('my name is '+this.name);
};

function ItBook(name, pages){
    this.name = name || 'itbook';
    this.pages = pages;
}

ItBook.prototype = new Book();
var ib = new ItBook('js', 300);
```

#### 通过Object.create继承
```javascript
var Book = {
    name: '',
    desc: function(){
        alert('my name is '+this.name);
    }
}

var ItBook = Object.create(Book);
ItBook.name = 'js';
```

#### 通过call重用代码
```javascript
function Book(name){
    this.name = name || '';
}
Book.prototype.desc = function(){
    alert('my name is '+this.name);
};

function ItBook(name, pages){
    Book.call(this, name);
    this.pages = pages;
}
```

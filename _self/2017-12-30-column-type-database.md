---
layout: post
title:  列式数据库
categories: nosql
---

## 什么是列式数据库

现在假设要存储用户信息，传统的面向行的关系型数据库表示大致是这样的：
id | username | realname | age | sex | status
----|----|----|----|----|----
1 | a | aa | 23 | 0 | 0
2 | b | null | null | 0 | 0

对于列式数据库来说，是这样的：


## 优缺点

## 数据结构
```
{
	"user_1" : {
		"profile" : {
			"username" : {
				"1" : "...",
				"2" : "..."
			},
			"realname" : "...",
			"age" : "...",
			"sex" : "...",
		},
		"status" : "0"
	},

	"user_2" : {
		"profile" : {
			"realname" : "...",
			"sex" : "...",
			"email" : "..."
		}
	}
}
```
user_1是行键、profile是列族，profile里的username是列，username里面的"1"、"2"是时间戳，status也是列

---
layout: post
category: "mysql"
title:  "SQL subquery 子查询"
---

### 非关联子查询

子查询可以单独执行

要查询所有参加了考试的学生，可以分解为两个步骤：

1. 先去score表得到参加了考试的学生
1. 根据studeng_id查询student表的学生信息

```sql
select * from student where id in (select student_id from score);
+----+------+------+-----+
| id | no   | name | sex |
+----+------+------+-----+
|  1 | S001 | S1   |   0 |
|  2 | S002 | S2   |   0 |
|  3 | S003 | S3   |   1 |
|  4 | S004 | S4   |   0 |
|  5 | S005 | S5   |   0 |
+----+------+------+-----+
```

```sql
select student.id, student.name,score.score from score, student where score.student_id=student.id and course_id=1 and score>=(select avg(score) from score where course_id=1);

+----+------+-------+
| id | name | score |
+----+------+-------+
|  5 | S5   |    88 |
+----+------+-------+
```



### 关联子查询

子查询依赖于外层查询

```sql
select st.name,(select score from score where student_id=st.id and course_id=1) as score from student st;
+------+-------+
| name | score |
+------+-------+
| S1   |  NULL |
| S2   |  NULL |
| S3   |  NULL |
| S4   |    76 |
| S5   |    88 |
| S6   |  NULL |
+------+-------+
```

查询参加了科目一考试的学生姓名
```sql
select st.name from student st where exists (select * from score sc where sc.student_id=st.id and sc.course_id=1);
+------+
| name |
+------+
| S4   |
| S5   |
+------+
```
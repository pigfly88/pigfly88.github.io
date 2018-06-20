## PHP函数
查出上个月最后一天的日期

```php
echo date('Y-m-d',strtotime('last day of last month'));
```

***
## 数据库
MySQL有哪些引擎？各有什么应用场景？
InnoDB什么情况锁表？
有一个表，每天100万的增量，如何设计？
设计索引的原则
有如下学生表、课程表、成绩表

```sql
CREATE TABLE `student` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `no` char(6) NOT NULL DEFAULT '' COMMENT '学号',
  `name` varchar(20) NOT NULL DEFAULT '' COMMENT '姓名',
  `sex` tinyint(1) unsigned NOT NULL DEFAULT '0' COMMENT '0：男，1：女',
  PRIMARY KEY (`id`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8 COMMENT '学生表';

CREATE TABLE `course` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(30) NOT NULL DEFAULT '' COMMENT '名称',
  PRIMARY KEY (`id`)
) ENGINE=MyISAM AUTO_INCREMENT=13 DEFAULT CHARSET=utf8 COMMENT '课程表';

CREATE TABLE `score` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键',
  `student_id` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '学生id',
  `course_id` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '课程id',
  `score` tinyint(3) unsigned NOT NULL DEFAULT '0' COMMENT '成绩',
  PRIMARY KEY (`id`),
  UNIQUE KEY `s_c` (`student_id`,`course_id`) USING HASH COMMENT '学生和课程唯一'
) ENGINE=InnoDB AUTO_INCREMENT=8 DEFAULT CHARSET=utf8 COMMENT '成绩表';
```


查出所有学生的学号、姓名、课程数、总分
```sql
select s.no, s.name, count(sc.course_id) as course_nums, SUM(score) as total_score
from student as s, score as sc
where sc.student_id=s.id
group by sc.student_id
```

查出没有全选课程的学生学号、姓名、课程数、总分
```sql
select s.no, s.name,count(sc.id) as course_count, sum(sc.score) as total_score from
student as s left join score as sc on s.id=sc.student_id
group by s.id having(course_count<3)
```

查出课程1比课程2得分高的学生
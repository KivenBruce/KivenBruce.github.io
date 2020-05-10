---
title: Oracle笔记-记一次有意思的sql查询结果集
date: 2018-11-01 11:46:40
tags: [note,database]
---

关键词: **not in (**),**group by()**

<!--more-->

## 一,in(),not in()的使用

我需要在查询结果集中排除一些结果;sql类似如下:

```sql
select * from table A where A.id not in(
    select id from table B where B.id>5
)
```

上面sql执行返回结果为空,直接查询select *  from table A,A.id为1,2,3,4,5

select id from table B的结果集为6,7,8,8,7,6,9,10....

为何最后没有返回呢?1,2,3,4,5符合not in(6,7,8,8,7,6,9,10)的啊.

执行如下sql,看看table B的id有多少种:

```sql
select distinct(id) from table B where B.id>5
```

发现返回的结果集为:6,7,8,9,null,10

**结果集中有null;**
首先说说Oracle中的null值:
null在oracle中代表未知，表示可能有，也可能没有。**任何与null值的普通运算都为null**，但可以用一些函数来处理null值，oracle排序中默认null最大。

我们可以理解为:

```sql
select * from table A where A.id in(1,2,null);
等价于
select * from table A where A.id = 1 or A.id = 2 or A.id = null;
```

```sql
select * from table A where A.id not in(1,2,null);
等价于
select * from table A where A.id!=1 and A.id!=2 and A.id!=null;
```

因为A.id=null 或者 A.id!=null的结果都是false,所以自然是没有返回结果集的;
**所以,我们在判断值是否为null的时候,一定不能这样写,得写成这样:A.id is null,A.id is not null**

**在使用not in查询的时候,必须注意结果集是否可能存在null值**,我们需要加上条件and B.id is not null;

## 二,max()/min()和group by的使用

比如有一张student表，和一张score表,字段如下:

student:

| id  | name |
| --- | ---- |
| 1   | A    |
| 2   | B    |
| 3   | C    |

score:

| id  | student_id | chinesse | english | math | title |
| --- | ---------- | -------- | ------- | ---- | ----- |
| 1   | 1          | 80       | 80      | 90   | 一模    |
| 2   | 2          | 80       | 90      | 90   | 一模    |
| 3   | 3          | 70       | 80      | 85   | 一模    |
| 4   | 1          | 85       | 85      | 80   | 二模    |
| 5   | 2          | 90       | 90      | 90   | 二模    |
| 6   | 3          | 80       | 90      | 70   | 二模    |

如果要查询两次考试总分最高的同学那次的考试成绩:
原sql:

```sql
SELECT 
    s.name,t.title, t.math, t.chinese, t.english, max(t.math+t.chinese+t.english) as total
FROM
    student s
        LEFT JOIN
    score t ON s.id = t.student_id
GROUP BY t.title
```

| name | title | math | chinese | english | total |
| ---- | ----- | ---- | ------- | ------- | ----- |
| A    | 一模    | 80   | 80      | 90      | 260   |
| A    | 二模    | 85   | 85      | 80      | 270   |

**从返回结果来看,只有total和title是对的,其他的都是按照t.title分组取的第一条数据;**

max()和min()都只影响()中的这个字段，因此无论是查英语还是数学的最大值，它都是对的，但它不会影响整行数据,group by默认返回每一组的第一条数据（每一组的数据排序都是按默认顺序排序的）;

那么既然group by默认返回每一组的第一条数据，我们是不是可以先排序再group by呢？

```sql
select * from (
    SELECT
 s.name,t.title, t.math, t.chinese, t.english, (t.math+t.chinese+t.english) as total
FROM
 student s
 LEFT JOIN
 score t ON s.id = t.student_id
 order by total desc
) r
group by r.title
```

这样是有对应关系的,**但是我发现只有MySQL支持这样的写法,Oracle不支持这样的写法,Oracle会报:ORA-00979:not a GROUP BY expression;Oracle的分组只能是查询分组的那个字段以及count或者sum操作**

```sql
select r.name,r.title,r.math,r.chinese,r.englist, r.total from (
  select s.name,t.title,t.math,t.chinese,t.englist,
    (t.math+t.chinese+t.english) as total,
    row_number() over (partition by t.title order by total desc)
   FROM
 student s
 LEFT JOIN
 score t ON s.id = t.student_id
) r where r.rn=1 
```

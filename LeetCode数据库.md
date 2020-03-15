---
title: 
   LeetCode数据库
tags: 
  [LeetCode,数据库]
categories:
	[SQL]
---

## [175. 组合两个表](https://leetcode-cn.com/problems/combine-two-tables/)

表1: `Person`

```java
+-------------+---------+
| 列名         | 类型     |
+-------------+---------+
| PersonId    | int     |
| FirstName   | varchar |
| LastName    | varchar |
+-------------+---------+
PersonId 是上表主键
```

表2: `Address`

```java
+-------------+---------+
| 列名         | 类型    |
+-------------+---------+
| AddressId   | int     |
| PersonId    | int     |
| City        | varchar |
| State       | varchar |
+-------------+---------+
AddressId 是上表主键
```


编写一个 SQL 查询，满足条件：无论 person 是否有地址信息，都需要基于上述两表提供 person 的以下信息：

```sql
FirstName, LastName, City, State
```

**解法一**

```sql
# Write your MySQL query statement below
select FirstName, LastName, City, State 
from Person p left join Address a 
on p.PersonId=a.PersonId;
```

## [596. 超过5名学生的课](https://leetcode-cn.com/problems/classes-more-than-5-students/)

有一个`courses` 表 ，有: **student (学生)** 和 **class (课程)**。

请列出所有超过或等于5名学生的课。

例如,表:

```java
+---------+------------+
| student | class      |
+---------+------------+
| A       | Math       |
| B       | English    |
| C       | Math       |
| D       | Biology    |
| E       | Math       |
| F       | Computer   |
| G       | Math       |
| H       | Math       |
| I       | Math       |
+---------+------------+
```


应该输出:

```java
+---------+
| class   |
+---------+
| Math    |
+---------+
```

**Note:**
学生在每个课中不应被重复计算。

**解法一**

```sql
select class from courses group by class having count(distinct student) >=5
```

## [176. 第二高的薪水](https://leetcode-cn.com/problems/second-highest-salary/)

编写一个 SQL 查询，获取 `Employee` 表中第二高的薪水（Salary） 。

```java
+----+--------+
| Id | Salary |
+----+--------+
| 1  | 100    |
| 2  | 200    |
| 3  | 300    |
+----+--------+
```


例如上述 `Employee` 表，SQL查询应该返回 `200` 作为第二高的薪水。如果不存在第二高的薪水，那么查询应返回 `null`。

```java
+---------------------+
| SecondHighestSalary |
+---------------------+
| 200                 |
+---------------------+
```

**解法一**

```sql
select 
(select distinct(Salary) from Employee order by Salary desc limit 1,1)
as SecondHighestSalary
```

**解法二**

```sql
select 
ifnull((select distinct(Salary) from Employee order by Salary desc limit 1,1),null)
as SecondHighestSalary
```


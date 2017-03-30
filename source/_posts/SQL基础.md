---
title: SQL基础
date: 2017-03-29 11:21:06
categories: 数据库
tags:
- SQL
---

### 一 概述

SQL是Structure Query Language的缩写，它是关系型数据库的应用语言。有IBM在20世纪70年代开发出来，作为IBM关系数据库原型System R的原型关系语言，实现了关系型数据库的信息检索。1986年，美国国家标准局（ANSI）制定出了SQL标准，目前绝大多数关系型数据库均支持SQL标准。本文的SQL语句是基于mysql的，有些语句在其他数据库上未必适用。

### 二 SQL语句

SQL语句不区分大小写，可以划分为3个类别：

- DDL语句：数据定义语言。用于定义数据段，数据库，表，列，索引等数据库对象，如create，drop，alter等
- DML语句：数据操纵语言。用于添加，删除，更新，查询数据库记录，并检查数据完整性，如insert，delete，update，select等
- DCL语句：数据控制语句。用于定义数据库，表，字段，用户等的访问权限和安全级别，如grant，revoke等

#### 1.DDL语句

##### 创建数据库

创建数据库

```bash
create database 数据库名;
```

使用数据库

```bash
use 数据库名;
```

展示一个数据库中所有的表

```bash
show tables;
```

##### 删除数据库
 
删除数据库

```bash
drop database 数据库名;
```

##### 创建表

创建表

```bash
create table emp(
	ename varchar(10),
	hiredate date,
	sal decimal(10,2)
	deptno int(2)
);
```

展示表信息

```bash
desc emp;
```

展示创建表的SQL语句信息

```bash
show create table emp \G;
```

##### 删除表（drop）

删除表

```bash
drop table emp;
```

##### 修改表（alter）

修改表字段类型

```bash
alter table emp modify ename varchar(20);
```

增加表字段

```bash
alter table emp add columu age int(3);
``` 

删除表字段

```bash
alter table emp drop columu age;
``` 

字段改名(mysql在标准SQL上的扩展)

```bash
alter table emp change age age1 int(4); //同时将字段类型改为int(4)
```

修改字段排列顺序first/after(mysql在标准SQL上的扩展)

```bash
alter table emp add birth date after ename;
```

```bash
alter table emp modify age int(3) first;
```

表改名

```bash
alter table emp rename emp1;
```

#### 2.DML语句

DML语句用于操作表中的数据。

##### 插入数据

```bash
insert into emp (ename,hiredate,sal,deptno) values ('zzx1','2000-01-01','2000',1);
//在数据库中单引号和双引号的效果相同
```

##### 更新数据

```bash
update emp set sal=4000 where ename='lisa'
```

更新多表

```bash
update emp a,dept b set a.sal=a.sal*b.deptno,b.deptname=a.ename where a.deptno=b.deptno;
```

##### 删除数据

```bash
delete from  emp where ename='lisa'
```

删除多表

```bash
delete a,b from emp a,dept b where a.deptno=b.deptno and a.deptno=3;
```

##### 查询数据

```bash
select * from emp
```

查询不重复的数据

```bash
select distinct deptno from emp
```

条件查询

```bash
select * from emp where deptno=1
```

```bash
select * from emp where deptno=1 and sal>=3000
```

##### 排序

```bash
select * from emp order by deptno;
```

限制显示条数（mysql特有，常用于分页操作）

```bash
select * from emp order by deptno limit 3;
```

##### 聚合

- 聚合函数：sum,count,max,min
- group by:表示要分类聚合的字段
- with rollup：可选，表示是否对分类聚合后的结果进行在汇总
- having：表示对分类聚合后的结果在进行条件查询（having与where不同，where是对聚前的数据进行条件过滤而having是聚合后，通常如果可以聚合前进行过滤优先考虑聚合前，这样需要聚合的集合更小，性能更高）

统计总人数

```bash
select count(1) from emp;
```

统计各部门人数

```bash
select deptno,count(1) from emp group by deptno;
```

统计各部门人数并汇总（汇总即是计算某一列所有数据的和）

```bash
select deptno,count(1) from emp group by deptno with rollup;
```

统计人数大于1的部门

```bash
select deptno,count(1) from emp group by deptno having count(1)>1;
```

##### 表连接

当需要同时显示多个表中的字段时，就可以使用表连接来实现，表连接分为内连接和外连接。

- 内连接：显示两张表中相匹配的数据
- 外连接：显示两张表中不匹配的数据

内连接（有一个特例自然连接，自动去除重复的记录）

```bash
select ename,deptname from emp,dept where emp.deptno=dept.deptno;
//ename是emp中的字段，deptname是dept中的字段
```

自然连接

```bash
select ename,deptname from emp join dept where emp.deptno=dept.deptno;
```

外连接

外连接又分为左连接和右连接

- 左连接：包含两表相匹配的列和左边表中不匹配的记录
- 右连接：包含两表中相匹配的列和右边表中不匹配的记录
- 全连接：返回两表中的所有记录
- 交叉连接：生成两表的笛卡尔积

左连接

```bash
select ename,deptname from emp left join dept on emp.deptno=dept.deptno;
```

右连接

```bash
select ename,deptname from emp right join dept on emp.deptno=dept.deptno;
```

全连接

```bash
select ename,deptname from emp outer join dept on emp.deptno=dept.deptno;
```

交叉连接

```bash
select ename,deptname from emp cross join dept on emp.deptno=dept.deptno;
```

##### 子查询

有时候当我们查询的时候需要的条件是另一个select语句的查询结果，这个时候就会用到子查询。子查询的关键字包括in，not in，=，！=，exits，not exits。

```bash
select * from emp where deptno in(select deptno from dept);
//如果子查询记录唯一应该用=代替，=的性能比in要好
```

有些子查询可以被表连接代替，如果可以被表连接代替使用表连接，表连接的性能比子查询要好。

##### 记录联合

有时候我们需要将两个select语句的查询结果合在一起显示，这是可以使用union和union all。

```bash
select deptno from union all select deptno from dept;
```

若需要去重，则

```bash
select deptno from union select deptno from dept;
```

#### DCL语句

一般是DBA使用，一般开发人员很少接触。

#### 查看帮助文档

可以使用“？ 关键字”的形式来查询某个关键字的用法，如果不知道有哪些命令可以使用“？ contents”来查询所有可供查询的分类。

### 三 mysql支持的数据类型

各个关系型数据库可能不同，我们应该了解所使用数据库支持的数据类型，这样在设计表时才能选择合适的数据类型。

### 四 函数

各个数据库可能不同，如字符串函数，数值函数，流程函数等

### 五 mysql存储引擎

插件式存储引擎是mysql最大的特性之一，它支持多种存储引擎。这之后会单独写个博客

### 六 字符集

各个数据库支持的不同

### 七 索引

这方面之后要单独写博客。

### 八 视图

视图是一张虚表，即并非真实存在的表。通常我们会将那些经常查询的数据生成视图，这样便于查询，但是视图中并没有数据，数据仍然在表中。

视图相对于普通表的优势有以下几点：

- 简单：使用视图的用户完全不需要关心后面对应的表结构，关联条件和筛选条件，视图就是已经过滤好的复合条件的结果集。
- 安全：可以只向用户提供所需查询的结果集的视图，而限制其对表结构的访问，保证了数据的安全。
- 数据独立：一旦视图已经生成了，对表增加列并不会影响到视图（如果改表名的话，需要更改视图）。

视图其实就是固化的一次select语句的结果集，之后每次查询这个结果集只需要查询视图即可。

##### 创建视图

```bash
create view staff_list_view as
select s.staff_id,s.first_name,s.last_name,s.address
from staff as s,address as a
where s.address_id=a.sddress_id;
```

mysql的视图定义有一些限制，如：在from关键字后面不能包含子查询，只能使用视图（将该子查询定义成视图，再查询视图生成视图）代替。

视图的可更新性与视图中查询的定义有关系，以下视图不可更新：

- 包含下列关键字的sql语句：sum,max,min,count,distinct,group by,having,union,union all等
- 常量视图
- select中包含子查询
- join
- from一个不能更新的视图
- where字句的子查询引用了from字句中的表

#####  删除视图

```bash
drop view staff_list_view;
```

##### 查看视图

与查看表相同

### 九 存储过程与函数

存储过程和函数都是事先经过编译并存储在数据库中的一段sql语句的集合。这两者不同的是函数有返回值而存储过程没有。
这里涉及到变量，条件，游标的使用，并且可使用流程函数（IF，CASE等）。这个在之后继续深入学习数据库时再详细介绍。

### 十 触发器

触发器也是sql语句的集合，但是它不需要手动执行，在达到某个条件时，它会自动执行相应的语句，即为触发。

### 十一 事务控制与锁定语句

锁，事务提交，回滚

### 十二 SQL中的安全问题

SQL注入。

### 十三 SQL Model及其相关问题






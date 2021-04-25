---
title: "MysqlDML"
date: 2021-04-25T11:10:37+08:00
lastmod: 2021-04-25T11:10:37+08:00
draft: false
keywords: []
description: "manjoc'blog"
tags: []
categories: []
author: "王清"
---

## DML

创建数据库时需要指定

`create database jumpserver default character set utf8 collate utf8_general_ci;`

# MySQL DML语句整理汇总

 更新时间：2019年04月25日 10:08:57  作者：s_pr1te  [![img](https://www.jb51.net/skin/2018/images/text-message.png) 我要评论](https://www.jb51.net/article/160218.htm#comments)

这篇文章主要介绍了MySQL DML语句整理汇总，文中通过示例代码介绍的非常详细，对大家的学习或者工作具有一定的参考学习价值，需要的朋友们下面随着小编来一起学习学习吧

DML操作是指对数据库中表记录的操作，主要包括表记录的插入（insert）、更新（update）、删除（delete）和查询（select），是开发人员日常使用最频繁的操作。

**1.插入(insert)**

格式1.

```
INSERT` `INTO` `emp(ename,hiredate,sal,deptno) ``VALUES``(``'zzx1'``,``'2000-01-01'``,``'2000'``,1);
```

格式2.

```
INSERT` `INTO` `emp ``VALUES``(``'lisa'``,``'2003-02-01'``,``'3000'``,2);
```

不用指定字段名称,但是value值需与表字段严格对应.

格式3.

```
INSERT` `INTO` `emp(ename,sal) ``VALUES``(``'dony'``,1000);
```

对于含可空字段、非空但是含有默认值的字段、自增字段，可以不用在INSERT后的字段列表里面出现，VALUES后面只写对应字段名称的VALUE：

格式4

在MySQL中，INSERT语句还可以一次性插入多条记录：

```
INSERT` `INTO` `tablename(field1,field2,......fieldn)``VALUES``(record1_value1,record1_value2,......record1_valuen),``(record2_value1,record2_value2,......record2_valuen),``......``(recordn_value1,recordn_value2,......recordn_valuen);
```

**2.更新(update)**

格式1

```
UPDATE` `emp ``SET` `sal=4000 ``WHERE` `ename=``'lisa'``;
```

格式2

```
UPDATE` `t1,t2,...tn ``SET``t1.field1=expr1,``t2.field2=expr2,``...``tn.fieldn=exprn [``WHERE` `CONDITION];
```

同时更新多个表中的数据

**3.删除(delete)**

格式1

```
DELETE` `FROM` `tablename [``WHERE` `CONDITION];
```

格式2

```
DELETE` `t1,t2,...tn ``FROM` `t1,t2,...tn [``WHERE` `CONDITION];
```

注意：不管是单表还是多表，不加where条件将会把表的所有记录删除，所以操作时一定要小心。

**4.查询(select)**

基本语法:

```
SELECT` `* ``FROM` `tablename [``WHERE` `CONDITION];
```

1.查询不重复的记录，可以用distinct关键字实现：

```
SELECT` `DISTINCT` `deptno ``FROM` `emp;
```

2.条件查询:

```
SELECT` `* ``FROM` `emp ``WHERE` `deptno=1;
```

其中，where 后面的条件除“=” 外，还可以使用 >、<、>=、<=、!=等比较运算符；

多个条件之间还可以使用or、and等逻辑运算符进行多条件联合查询

3.排序和限制:

```
SELECT` `* ``FROM` `tablename [``WHERE` `CONDITION] [``ORDER` `BY` `field1 [``DESC``|``ASC``], field2 [``DESC``|``ASC``],...fieldn [``DESC``|``ASC``]];
```

其中，DESC和ASC是排序关键字，
DESC表示按照字段进行降序排列（上大-下小），
ASC则表示升序排列（上小-下大），
如果不写此关键字默认是升序排列。
ORDER BY 后面可以跟多个不同的排列字段，并且每个排序字段可以有不同的排序顺序。

如果排序字段的值一样，则值相同的字段按照第二个排序字段进行排序，以此类推。如果只有一个排序字段，则这字段相同的记录将会无序排列。

```
SELECT` `......[LIMIT offset_start,row_count];
```

对于排序后的记录，如果希望只显示一部分，而不是全部，这时，就可以使用LIMIT关键字来实现.

LIMIT 经常和ORDER BY一起配合使用来进行记录的分页显示。

注意：limit属于MySQL扩展SQL92后的语法，在其他数据库上并不能通用。

4.聚合

很多情况下，我们需要进行一些汇总操作，比如统计整个公司的人数或者统计每个部门的人数，这是就要用到SQL的集合操作。

```
SELECT` `[field1,field2,......fieldn] fun_name``FROM` `tablename``[``WHERE` `where_contition]``[``GROUP` `BY` `field1,field2,......fieldn``[``WITH` `ROLLUP``]]``[``HAVING` `where_contition];
```

对其参数进行以下说明：

1. fun_name 表示要做的集合操作，也就是聚合函数，常用的有sum（求和）、count(*)（记录数）、max（最大值）、min（最小值）。
2. GROUP BY 关键字表示要进行分类聚合的字段，比如要按照部门分类统计员工数量，部门就应该写在group by后面。
3. WITH ROLLUP 是可选语法，表明是否对分类聚合后的结果进行再汇总。
4. HAVING 关键字表示对分类后的结果再进行条件的过滤。

注意：having和where的区别在于having是对聚合后的结果进行条件的过滤，而where是在聚合前就对记录进行过滤，如果逻辑允许，我们尽可能用where先过滤记录，这样因为结果集减小，将对聚合的效率大大提高，最后再根据逻辑看是否用having进行再过滤。

**5.表连接**

表连接分为内连接和外连接，它们之间的最主要区别是：
内连接仅选出两张表中互相匹配的记录，
而外连接会选出其他不匹配的记录。我们常用的是内连接。

外连接有分为左连接和右连接，具体定义如下：

1. 左连接：包含所有的左边表中的记录甚至是右边表中没有和它匹配的记录
2. 右连接：包含所有的右边表中的记录甚至是左边表中没有和它匹配的记录

```
SELECT` `ename,deptname ``FROM` `emp ``LEFT` `JOIN` `dept ``ON` `emp.deptno=dept.deptno;
```

 **6.子查询**

某些情况，当我们查询的时候，需要的条件是另一个SELECT语句的结果，这个时候，就要用到子查询。
用于子查询的关键字主要包括in、not in、=、!=、exists、not exists等。

```
SELECT` `* ``FROM` `emp ``WHERE` `deptno ``IN``(``SELECT` `deptno ``FROM` `dept);
```

注意：子查询和表连接之间的转换主要应用在两个方面：

MySQL 4.1 以前的版本不支持子查询，需要用表连接来实现子查询的功能
表连接在很多情况下用于优化子查询

**7.记录联合**

我们经常会碰到这样的应用，将两个表的数据按照一定的查询条件查询出来后，将结果合并到一起显示出来，这个时候，就需要用到union和union all关键字来实现这样的功能，具体语法如下：

```
SELECT` `* ``FROM` `t1``UNION``|``UNION` `ALL``SELECT` `* ``FROM` `t2``......``UNION``|``UNION` `ALL``SELECT` `* ``FROM` `tn;
```

UNION 和 UNION ALL的主要区别是 UNION ALL 是把结果集直接合并在一起，而UNION是将UNION ALL后的结果进行一次DISTINCT，去除重复记录后的结果。

以上所述是小编给大家介绍的MySQL DML语句整理汇总详解整合，希望对大家有所帮助，如果大家有任何疑问请给我留言，小编会及时回复大家的。在此也非常感谢大家对脚本之家网站的支持！

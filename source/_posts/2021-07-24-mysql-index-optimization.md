---
title: MySQL索引详解
date: 2021-07-24 23:47:20
tags: 
- MySQL
- 索引
- 数据库
- 数据库优化
categories: 
- 数据库
- MySQL修炼
- 索引
---

## 前言

> 索引的建立对于MySQL的高效运行是很重要的，索引可以大大提高MySQL的检索速度。

我最近在一个工单系统的迭代中遇到了性能问题，由于匆忙上线了个人的待办和待认领功能，导致数据库的查询难度激增，所以需要从数据库优化的方向来考虑，分为**查询优化**、**索引优化**与**表结构拆分重构**。我打算先从最简单高效的索引入手。

所谓索引就是为特定的 MySQL 字段进行一些特定的算法排序，比如二叉树的算法和哈希算法，哈希算法是通过建立特征值，然后根据特征值来快速查找。而用的最多，并且是 MySQL 默认的就是二叉树算法 BTREE，通过BTREE算法建立索引的字段，比如扫描20行就能得到未使用BTREE前扫描了2^20行的结果。

去年我也有一篇文章有提到 MySQL 索引相关的内容，现在看来不太全面，所以打算系统又全面地写一写索引和性能优化的内容。

<!-- more -->

## 为什么要用索引

索引就是根据表中的一列或若干列按照一定顺序建立的列值与记录行之间的对应关系表，实质上是一张描述索引列的列值与原表中记录行之间一 一对应关系的有序表。

索引是 MySQL 中十分重要的数据库对象，是数据库性能调优技术的基础，常用于实现数据的快速检索。

在 MySQL 中，通常有以下两种方式访问数据库表的行数据：

###  顺序访问

顺序访问是在表中实行全表扫描，从头到尾逐行遍历，直到在无序的行数据中找到符合条件的目标数据。

顺序访问实现比较简单，但是当表中有大量数据的时候，效率非常低下。例如，在几千万条数据中查找少量的数据时，使用顺序访问方式将会遍历所有的数据，花费大量的时间，显然会影响数据库的处理性能。

###  索引访问

索引访问是通过遍历索引来直接访问表中记录行的方式。

使用这种方式的前提是对表建立一个索引，在列上创建了索引之后，查找数据时可以直接根据该列上的索引找到对应记录行的位置，从而快捷地查找到数据。索引存储了指定列数据值的指针，根据指定的排序顺序对这些指针排序。

例如，在学生基本信息表 tb_students 中，如果基于 student_id 建立了索引，系统就建立了一张索引列到实际记录的映射表。当用户需要查找 student_id 为 12022 的数据的时候，系统先在 student_id 索引上找到该记录，然后通过映射表直接找到数据行，并且返回该行数据。因为扫描索引的速度一般远远大于扫描实际数据行的速度，所以采用索引的方式可以大大提高数据库的工作效率。

简而言之，不使用索引，MySQL 就必须从第一条记录开始读完整个表，直到找出相关的行。表越大，查询数据所花费的时间就越多。如果表中查询的列有一个索引，MySQL 就能快速到达一个位置去搜索数据文件，而不必查看所有数据，这样将会节省很大一部分时间。

## 索引的优缺点

索引有其明显的优势，也有其不可避免的缺点。

### 优点

索引的优点如下：

- 通过创建唯一索引可以保证数据库表中每一行数据的唯一性。
- 可以给所有的 MySQL 列类型设置索引。
- 可以大大加快数据的查询速度，这是使用索引最主要的原因。
- 在实现数据的参考完整性方面可以加速表与表之间的连接。
- 在使用分组和排序子句进行数据查询时也可以显著减少查询中分组和排序的时间

### 缺点

增加索引也有许多不利的方面，主要如下：

- 创建和维护索引组要耗费时间，并且随着数据量的增加所耗费的时间也会增加。
- 索引需要占磁盘空间，除了数据表占数据空间以外，每一个索引还要占一定的物理空间。如果有大量的索引，索引文件可能比数据文件更快达到最大文件尺寸。
- 当对表中的数据进行增加、删除和修改的时候，索引也要动态维护，这样就降低了数据的维护速度。

## 索引的类型

**UNIQUE唯一索引**

不可以出现相同的值，可以有NULL值。

**INDEX普通索引**

允许出现相同的索引内容。

**PRIMARY KEY主键索引**

不允许出现相同的值，且不能为NULL值，一个表只能有一个primary_key索引。

**fulltext index 全文索引**

上述三种索引都是针对列的值发挥作用，但全文索引，可以针对值中的某个单词，比如一篇文章中的某个词，然而并没有什么卵用，因为只有myisam以及英文支持，并且效率让人不敢恭维，但是可以用coreseek和xunsearch等第三方应用来完成这个需求。

除了以上四种单列索引，即一个索引只包含单个列，一个表可以有多个单列索引。还有复合索引，即一个索引包含多个列。

## 索引的CURD

**索引的创建**

ALTER TABLE

适用于表创建完毕之后再添加。

ALTER TABLE 表名 ADD 索引类型 (unique,primary key,fulltext,index)(索引名)(字段名)

```sql
ALTER TABLE `table_name` ADD INDEX `index_name` (`column_list`) -- 索引名，可要可不要；如果不要，当前的索引名就是该字段名。 
ALTER TABLE `table_name` ADD UNIQUE (`column_list`) 
ALTER TABLE `table_name` ADD PRIMARY KEY (`column_list`) 
ALTER TABLE `table_name` ADD FULLTEXT KEY (`column_list`)
```

CREATE INDEX

CREATE INDEX可对表增加普通索引或UNIQUE索引。

```sql
-- 例：只能添加这两种索引 
CREATE INDEX index_name ON table_name (column_list) 
CREATE UNIQUE INDEX index_name ON table_name (column_list)
-- 复合索引
CREATE INDEX index_name ON table_name (column_list1， column_list2) 
```

**另外，还可以在建表时添加：**

```sql
CREATE TABLE `test1` ( 
  `id` smallint(5) UNSIGNED AUTO_INCREMENT NOT NULL, -- 注意，下面创建了主键索引，这里就不用创建了 
  `username` varchar(64) NOT NULL COMMENT '用户名', 
  `nickname` varchar(50) NOT NULL COMMENT '昵称/姓名', 
  `intro` text, 
  PRIMARY KEY (`id`),  
  UNIQUE KEY `unique1` (`username`), -- 索引名称，可要可不要，不要就是和列名一样 
  KEY `index1` (`nickname`), 
  FULLTEXT KEY `intro` (`intro`) 
) ENGINE=MyISAM AUTO_INCREMENT=4 DEFAULT CHARSET=utf8 COMMENT='后台用户表';
```

**索引的删除**

```sql
DROP INDEX `index_name` ON `talbe_name`  
ALTER TABLE `table_name` DROP INDEX `index_name` 
-- 这两句都是等价的,都是删除掉table_name中的索引index_name; 

ALTER TABLE `table_name` DROP PRIMARY KEY -- 删除主键索引，注意主键索引只能用这种方式删除
```

**索引的查看**

```sql
show index from tablename;
```

**索引的更改**

更改个毛线，删掉重建一个就好

## 创建索引的技巧

1. 维度高的列创建索引。

2. 数据列中**不重复值**出现的个数，这个数量越高，维度就越高。
   - 如数据表中存在8行数据a,b ,c,d,a,b,c,d这个表的维度为4。
   - 要为维度高的列创建索引，如性别和年龄，那年龄的维度就高于性别。
   - 性别这样的列不适合创建索引，因为维度过低。

3. 对 where,on,group by,order by 中出现的列使用索引。

4. 对较小的数据列使用索引，这样会使索引文件更小，同时内存中也可以装载更多的索引键。

5. 为较长的字符串使用前缀索引。

   - 当要索引的列字符很多时 索引则会很大且变慢
   - 使用字符串的前n位当做索引，如`ALTER TABLE x_test ADD INDEX (x_name(n))`

6. 不要过多创建索引，除了增加额外的磁盘空间外，对于DML操作的速度影响很大，因为其每增删改一次就得从新建立索引。

7. 使用组合索引，可以减少文件索引大小，在使用时速度要优于多个单列索引。

## 什么是聚集索引和非聚集索引？

我们知道 Mysql 底层是用 B+ 树来存储索引的，且数据都存在叶子节点。对于 InnoDB 来说，它的主键索引和行记录是存储在一起的，因此叫做聚集索引（clustered index)，也可以叫聚簇索引。

除了聚集索引，其它索引都叫做非聚集索引（secondary index）。包括普通索引，唯一索引等。

另外需要注意，在 InnoDB 中有且只有一个聚集索引。它有三种情况：

- 若表存在主键，则主键索引就是聚集索引。
- 若不存在主键，则会把第一个非空的**唯一索引**作为聚集索引。
- 否则，就会隐式的定义一个 rowid 作为聚集索引。

在 InnoDB 中，主键索引的叶子节点存储的是主键和行记录，而普通索引的叶子节点存储的是主键。

因此，id 为聚集索引，name 为非聚集索引。它们对应的 B+ 树结构如下图所示，

{% asset_img clustered-secondary-index.jpg %}

## 什么是回表查询？

从上边的索引存储结构，我们可以看到，在主键索引树上，通过主键就可以一次性查出来我们所需要的数据，速度非常的快。

因为主键和行记录就存储在一起，定位到了主键，也就定位到了所要找的记录，当前行的所有字段都在这（这也是为什么我们说，在创建表的时候，最好是创建一个主键，查询时也尽量用主键来查询）。

对于普通索引，如例子中的 name，则需要根据 name 的索引树（非聚集索引）找到叶子节点对应的主键，然后再通过主键去主键索引树查询一遍，才可以得到要找的记录。这就叫 **回表查询**。

以如下 sql 为例。

```sql
select * from student where name='zs';
```

它需要查询两遍索引树。

- 通过非聚集索引定位到主键 id=1。
- 通过聚集索引定位到主键id为1，对应的行记录。

{% asset_img backup-table-query.jpg %}

### 什么是索引覆盖？

对于上边的回表查询来说，无疑会降低查询效率。那么，有没有什么办法，让它不回表呢？

答案当然是有啦，就是**索引覆盖**。

所谓索引覆盖，就是在用这个索引查询时，使它的索引树，查询到的叶子节点上的数据可以覆盖到你查询的所有字段，这样就可以避免回表。

**举个栗子：**

创建联合索引如下，

```sql
CREATE INDEX name_age_inde ON `idx_stu` (`name`,`age`)
```

利用联合索引查询如下：

```sql
-- 覆盖联合索引中的字段
SELECT id,name,age FROM student WHERE name='zs' AND age=12; 
```

这样，当查询索引树的时候，由于SELECT的字段全部在联合索引**(zs, 12)**上，就不需要使用主键（聚簇索引）定位这条记录以便以获取SELECT字段。

{% asset_img covering-index.jpg %}

## 最后

以上就是笔者整理出的索引的概念、优势和技巧等内容，后续会单独分析复合索引（组合索引）的特性，以及在数据库实例中，索引优化该怎么上手。



------

参考资料：

[MySQL索引及优化 | Sapphire (notspr.com)](https://notspr.com/2020/10/05/mysql-basis/)

[MySQL 索引优化全攻略 | 菜鸟教程 (runoob.com)](https://www.runoob.com/w3cnote/mysql-index.html)

[CS-Notes/MySQL.md at master · CyC2018/CS-Notes (github.com)](https://github.com/CyC2018/CS-Notes/blob/master/notes/MySQL.md#mysql-索引)

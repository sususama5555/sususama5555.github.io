---
title: MySQL索引 - 优化步骤
date: 2021-08-21 19:16:58
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
# 前言

继上一篇文章，继续补充MySQL索引的优化步骤

{% post_link mysql-index-optimization-1 返回上一篇《MySQL索引 - 理论详解》 %}

在日常的后端开发中，离不开对数据库的增删改查，往往在查询性能指标、操作日志这种大表时，会遇到数据库瓶颈，优化的方法有很多，例如水平拆分、垂直拆分等，其中最简单有效、代价最低的方法就是利用索引。

先来精炼复习下索引原理。

<!-- more -->

# 索引原理

不同存储引擎具有不同的索引类型和实现。大多数 MySQL 引擎的默认索引类型是基于B+树这种数据结构实现的，例如我们在使用的InnoDB，以下的内容都是针对B+树索引来分析。

他的所有叶子节点位于同一层，而且叶子节点是顺序访问的，通过索引，我们不再需要进行全表扫描，只需要对树进行递归地二分查找即可，所以查找速度快很多。这里的内容很多，大家可以自己去查看文档。

{% asset_img index-traverse.png %}

我的理解是，数据库一张表相当于一本字典，收录的每一个字就是一行数据，书的目录就相当于索引。

如果没有目录，例如去查“奥”这个字，我们需要把每一页都检索一遍才能找到。

通过查找目录，我们可以根据“奥”的拼音来快速定位到页码，只需要检索一页就可以了。这个默认的目录就是聚簇索引，也就是主键索引，一般是id这个主键字段。

{% asset_img dictionary-index.png %}

```mysql
SELECT * FROM 字典 WHERE 拼音='ao'
```

但是如果想按笔画或者偏旁或者多个条件一次查找，就不能通过拼音目录来索引，我们需要额外建立索引，这种索引叫非聚集索引，分为普通索引，唯一索引。

在 InnoDB 中，**主键索引**的叶子节点存储的是主键和行记录，而**普通索引**的叶子节点存储的是主键的值。

利用普通索引二分查找到主键过后，然后再通过主键去主键索引树查询一遍，才可以得到要找的记录。这就叫 **回表查询**。

{% asset_img clustered-secondary-index.jpg %}

# 优化步骤

## 开启慢查询

mysql的慢查询功能，可以记录执行超过多少秒的sql语句，我们可以通过慢查询日志来知道哪些SQL很慢。

```mysql
-- 慢查询配置
SHOW VARIABLES like '%slow%';
SHOW VARIABLES like '%dir%';
```

## 使用 Explain 进行分析

EXPLAIN + 查询SQL

Explain 出来的信息有10列，分别是id、select_type、table、type、possible_keys、key、key_len、ref、rows、Extra

| 字段          | 解释                       |
| ------------- | -------------------------- |
| id            | 选择标识符                 |
| select_type   | 表示查询的类型             |
| table         | 输出结果集的表             |
| partitions    | 匹配的分区                 |
| type          | 表示表的连接类型           |
| possible_keys | 表示查询时，可能使用的索引 |
| key           | 表示实际使用的索引         |
| key_len       | 索引字段的长度             |
| ref           | 列与索引的比较             |
| rows          | 扫描出的行数(估算的行数)   |
| filtered      | 按表条件过滤的行百分比     |
| Extra         | 执行情况的描述和说明       |

比较重要的字段：

- select_type :  查询类型，有简单查询、联合查询、子查询等
- key :  使用的索引
- rows :  扫描的行数

## 索引CURD

### **索引的创建**

**ALTER TABLE**

适用于表创建完毕之后再添加。

ALTER TABLE 表名 ADD 索引类型 (`unique`、`primary key`、`fulltext`、`index`)(索引名)(字段名)

```mysql
ALTER TABLE `table_name` ADD INDEX `index_name` (`column_list`) -- 索引名，可要可不要；如果不要，当前的索引名就是该字段名。 
ALTER TABLE `table_name` ADD UNIQUE (`column_list`) 
ALTER TABLE `table_name` ADD PRIMARY KEY (`column_list`) 
ALTER TABLE `table_name` ADD FULLTEXT KEY (`column_list`)
```

**CREATE INDEX**

CREATE INDEX可对表增加普通索引或UNIQUE索引。

```mysql
-- 例：只能添加这两种索引 
CREATE INDEX index_name ON table_name (column_list) 
CREATE UNIQUE INDEX index_name ON table_name (column_list)
-- 复合索引
CREATE INDEX index_name ON table_name (column_list1， column_list2)
-- 前缀索引
CREATE INDEX index_name ON table_name (column_list(n)) -- 前n位做索引
```

**另外，还可以在建表时添加**

```mysql
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

### **索引的删除**

```mysql
DROP INDEX `index_name` ON `talbe_name`  
ALTER TABLE `table_name` DROP INDEX `index_name` 
-- 这两句都是等价的，都是删除掉table_name中的索引index_name; 

ALTER TABLE `table_name` DROP PRIMARY KEY -- 删除主键索引，注意主键索引只能用这种方式删除
```

### **索引的查看**

```mysql
SHOW INDEX FROM tablename;
```

### **索引的更改**

更改个毛线，删掉重建一个就好

# 创建索引的技巧

1. 对维度高的列创建索引。
2. 数据列中**不重复值**出现的个数，这个数量越高，维度就越高。
   - 如数据表中存在8行数据，例如a,b,c,d,a,b,c,d这个列的维度为4。
   - 要为维度高的列创建索引，如性别和年龄，那年龄的维度就高于性别。
   - 性别这样的列不适合创建索引，因为维度过低。
3. 对 where、on、group by、order by 中出现的列使用索引。
4. 对较小的数据列使用索引，这样会使索引文件更小，同时内存中也可以装载更多的索引键。
5. 为较长的字符串使用前缀索引。
   - 当要索引的列字符很多时 索引则会很大且变慢
   - 使用字符串的前n位当做索引，如`ALTER TABLE x_test ADD INDEX (x_name(n))`
6. 不要过多创建索引，除了增加额外的磁盘空间外，对于DML操作的速度影响很大，因为其每增删改一次就得从新建立索引。
7. 使用组合索引，可以减少文件索引大小，在使用时速度要优于多个单列索引。

# 组合索引

按覆盖列的数量，索引分**单列索引**和**组合索引**。单列索引，即一个索引只包含单个列，一个表可以有多个单列索引，但这不是组合索引。

***组合索引（Composite Index）***，又称复合索引、联合索引、多列索引，是建立在多个列上的索引，适用在多个列一起使用或者是从左到右方向部分连续列一起使用的业务场景。

```mysql
ALTER TABLE `table_name` ADD INDEX (`col1`,`col2`,`col3`);
```

当我们创建(**col1**,**col2**,**col3**)组合索引时，

相当于创建了**(col)单列索引**，**(col1,col2)组合索引**以及**(col1,col2,col3)组合索引**，想要索引生效，只能使用**col1**和**col1,col2**和**col1,col2,col3**三种组合；当然，**col1,col3组合也可以，但实际上只用到了col1的索引，col3并没有用到！**

组合索引是有明确的`先后顺序`的，满足组合索引的**最左前缀匹配原则**。

## 优势

### 一个顶三个

建了一个(a,b,c)的复合索引，那么实际等于建了(a),(a,b),(a,b,c)三个索引，因为每多一个索引，都会增加写操作的开销和磁盘空间的开销。对于大量数据的表，这是不小的开销。

### 覆盖索引

如果有组合索引(a,b,c)，如果有如下的sql（`select的字段全部命中在组合索引上`）：

```mysql
SELECT a, b, c FROM TABLE WHERE a=1 AND b = 1;
```

那么MySQL可以直接通过遍历索引取得数据，而无需回表查找，这减少了很多的随机io操作。减少io操作，特别的随机 io 其实是dba主要的优化策略。在真正的实际应用中，覆盖索引是主要提升性能的优化手段之一。

### 索引列越多，通过索引筛选出的数据越少

索引列越多，通过索引筛选出的数据越少。有1000W条数据的表，有如下sql：

```mysql
SELECT * FROM TABLE WHERE a = 1 AND b =2 AND c = 3;
```

假设假设每个条件可以筛选出10%的数据，

如果只有单值索引，那么通过该索引能筛选出1000w * 10% = 100w 条数据，然后再回表从100w条数据中找到符合b=2 and c= 3的数据，然后再排序，再分页；

如果是组合索引，通过索引筛选出1000w * 10% * 10% *10% = 1w，然后再排序、分页，哪个更高效，一眼便知。

概况：**多个单列索引在多条件查询时只会生效第一个索引，所以多条件联合查询时最好建组合索引！**

## 最左前缀匹配原则

> 在MySQL建立组合索引时会遵守最左前缀匹配原则，即最左优先，在检索数据时从联合索引的最左边开始匹配。

首先，最左前缀原则是发生在复合索引上的，只有复合索引才会有所谓的左和右之分。

要想理解联合索引的最左匹配原则，先来理解下索引的底层原理。索引的底层是一颗B+树，那么联合索引的底层也就是一颗B+树，只不过联合索引的B+树节点中存储的是键值。由于构建一棵B+树只能根据一个值来确定索引关系，所以数据库依赖联合索引最左的字段来构建。

下图是一个（a,b）的联合索引的索引树：

[![img](https://notspr.com/2021/07/25/mysql-composite-index/composite-index-btree.png)](https://notspr.com/2021/07/25/mysql-composite-index/composite-index-btree.png)

其中a是有顺序的，而b是没有顺序的，但是在a等值的情况下，b值又是按顺序排列的，这种顺序是相对的。

这是因为MySQL创建联合索引时首先对联合索引的最左边字段排序，在最左字段的排序基础上，然后在对次左字段进行排序。所以直接查询非最左字段无法利用该联合索引。

对于这个结论，我们可以通过`EXPLAIN`语句来快速验证。

# Django中集成索引

```python
class TicketEventLog(models.Model):

	# 单列索引 -> db_index=True
    sn = models.CharField(_("单据号"), max_length=LEN_NORMAL, db_index=True)
    # --------------
    # ---- 字段 -----
    # --------------
    processors_type = models.CharField(_("处理人类型"), max_length=LEN_SHORT, choices=PROCESSOR_CHOICES, default="OPEN")
    processors = models.CharField(_("处理人列表"), max_length=LEN_LONG, default=EMPTY_STRING, null=True, blank=True)

    class Meta:
        # 单列索引
        indexes = (models.Index(fields=["sn"]), )
        # 组合索引 -> index_together
        index_together = ("operator", "operate_at")
        # 联合约束 -> unique_together
        unique_together = ("ticket_id", "role_id")

```

## ATTENTION！

Django的 contain 和 icontain 不会走索引：

```python
TicketEventLog.object.filter(processors__contain="ryanshu")
```

相当于sql：

```mysql
`ticket_ticketeventlog`.`operator` LIKE '%ryanshu%'
```

这种前缀模糊匹配，"%"通配符开头的情况，索引失效。

可以使用 startswith 和 istartswith 替换一部分 contain 的 orm语句。
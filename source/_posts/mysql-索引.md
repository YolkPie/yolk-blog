---
title: mysql 索引
date: 2020-02-04 16:34:22
tags: 
- mysql
- 索引
categories: Java
author: 远超
keywords: mysql,索引
description: mysql 索引
cover: https://img14.360buyimg.com/imagetools/jfs/t1/99941/31/11809/29824/5e392d51Efbe7ca53/acc64d7b5744c591.jpg
top_img: https://img11.360buyimg.com/imagetools/jfs/t1/98677/14/11560/34956/5e392d22E4268c3e9/fae1ce3b5325a62a.jpg
---
## 索引

索引(key)是存储引擎用于快速找到记录的一种数据结构。它和一本书中目录的工作方式类似——当要查找一行记录时，先在索引中快速找到行所在的位置信息，然后再直接获取到那行记录。
在MySql中，索引是在存储引擎层而不是服务器层实现的，所以不同的存储引擎对索引的实现和支持都不相同。
<!-- more -->

## 索引的优点

- 索引大大减少了服务器需要扫描的数据量
- 索引可以帮助服务器避免排序和临时表
- 索引可以将随机I/O变为顺序I/O

## 索引类型

- 普通索引
- UNIQUE索引
- 主键索引

## MyISAM索引与InnoDB索引那个快

### MyISAM索引的实现

      MyISAM索引文件和数据文件是分离的，索引文件仅保存记录所在页的指针（物理位置），通过这些地址来读取页，进而读取被索引的行。下图是MyISAM的索引原理图：（为了简化，一个页内只存放了两条记录。）

![](mq1.png)
      上图所提供的示例表字段有Col1（ID）、Col2(age)、Col3（name）三个，其中Col1为Primary Key（主键），上图很好地说明了树中叶子保存的是对应行的物理位置。通过该值，存储引擎能顺利地进行回表查询，得到一行完整记录。同时，每个叶子页也保存了指向下一个叶子页的指针。从而方便叶子节点的范围遍历。
    
      而对于二级索引，在 MyISAM存储引擎中以与上图同样的方式实现，可以看出MyISAM的索引文件仅仅保存数据记录的地址。

### InnoDB索引的实现

1. 1)聚集索引

      与 MyISAM相同的一点是，InnoDB 也采用 B+Tree这种数据结构来实现 B-Tree索引。而很大的区别在于，InnoDB 存储引擎采用&quot;聚集索引&quot;的数据存储方式实现B-Tree索引，所谓&quot;聚集&quot;，就是指数据行和相邻的键值紧凑地存储在一起，注意 InnoDB 只能聚集一个叶子页（16K）的记录（即聚集索引满足一定的范围的记录），因此包含相邻键值的记录可能会相距甚远。

注意: innodb来说,

1 主键索引 既存储索引值,又在叶子中存储行的数据

2 如果没有主键, 则会Unique key做主键

3 如果没有unique,则系统生成一个内部的rowid做主键.

4 像innodb中,主键的索引结构中,既存储了主键值,又存储了行数据,这种结构称为&quot;聚簇索引&quot;

下图说明了 InnoDB聚集索引的实现方式，同时也体现了一张 innoDB表的结构，可以看到，InnoDB 中，主键索引和数据是一体的，没有分开。：

![](mq2.png)
这种实现方式，给予了 InnoDB 按主键检索的超高性能。可以有目的性地选择聚集索引，比如一个邮件表，可以选择用户ID来聚集数据，这样只需要从磁盘读取较少并且连续的数据页就能获得某个id的用户全部的邮件，避免了读取分散页时所耗费的随机I/O。

2)辅助索引

      而对于辅助索引，InnoDB采用的方式是在叶子页中保存主键值，通过这个主键值来回表（上图）查询到一条完整记录，因此按辅助索引检索实际上进行了二次查询，效率肯定是没有按照主键检索高的。下图是辅助索引的实现方式：

![](mq3.png)
      由于每个辅助索引都包含主键索引，因此，为了减小辅助索引所占空间，我们通常希望 InnoDB 表中的主键索引尽量定义得小一些（值得一提的是，MySIAM会使用前缀压缩技术使得索引变小，而InnoDB按照原数据格式进行存储。），并且希望InnoDB的主键是自增长的，因为如果主键并非自增长，插入时，由于写入时乱序的，会使得插入效率变低。

参考：https://blog.csdn.net/frycn/article/details/70158313

https://blog.csdn.net/ljfphp/article/details/80029968

5. 索引:用于加快查找数据的速度的一种内部机制。

如何定位一个慢sql：

1. 开启慢查询日志，默认是不开启，showvariableslike&#39;%slow%、set log\_slow\_queries on  set long\_query\_time=5; 默认是10
2. mysqldumpslow --help可显示其参数的使用

得到返回记录最多的20个sql

mysqldumpslow -s r -t 20 sqlslow.log

得到平均访问次数最多的20条sql

mysqldumpslow -s ar -t 20 sqlslow.log

参考：[https://blog.csdn.net/sunyuhua\_keyboard/article/details/81204020](https://blog.csdn.net/sunyuhua_keyboard/article/details/81204020)

1. explain + 慢sql
2. show profiles 细致分析mysql 生命周期和执行细节
3. sql 服务器调优（后边有介绍）

先讲解explain 参考：[https://www.cnblogs.com/yycc/p/7338894.html](https://www.cnblogs.com/yycc/p/7338894.html)

https://www.cnblogs.com/gomysql/p/3720123.html

![](mq4.png)
       

EXPLAIN select \* from store\_info where id in (SELECT store\_info\_id from attachment where file\_type = 2);

![](mq5.png)
Id：这是SELECT的查询序列号，id相同时 由上至下执行，id不同时，数值越大越先执行。

select\_type: SELECT类型,可以为以下任何一种:

- **SIMPLE**:简单SELECT(不使用UNION或子查询)
- **PRIMARY**:最外面的SELECT
- **UNION**:UNION中的第二个或后面的SELECT语句
- **DEPENDENT UNION**:UNION中的第二个或后面的SELECT语句,取决于外面的查询
- **UNION RESULT**:UNION 的结果
- **SUBQUERY**:子查询中的第一个SELECT
- **DEPENDENT SUBQUERY**:子查询中的第一个SELECT,取决于外面的查询
- **DERIVED**:导出表的SELECT(FROM子句的子查询)

Table ：输出的行所引用的表

Type: 访问类型  ALL, index,  range, ref, eq\_ref, const, system, NULL

- ==system==: 表仅有一行(=系统表)。这是const联接类型的一个特例。
- ==const==: 表最多有一个匹配行,它将在查询开始时被读取。因为仅有一行,在这行的列值可被优化器剩余部分认为是常数。const表很快,因为它们只读取一次!

     ```sql
     explain select\*from ( select\*from t1 where id= **1** )b1;
     ```

- **** eq\_ref:就在使用的索引是唯一索引，对于每个索引键值，表中只有一条记录匹配，简单来说，就是多表连接中使用primary key或者 unique key作为关联条
- **** ref: 使用非唯一索引扫描或者唯一索引的前缀扫描，返回匹配某个单独值的记录行
- **** range:索引范围扫描，对索引的扫描开始于某一点，返回匹配值域的行。显而易见的索引范围扫描是带有between或者where子句里带有\&lt;, \&gt;查询。当mysql使用索引去查找一系列值时，例如IN()和OR列表，也会显示range（范围扫描）,当然性能上面是有差异的。
- **** index:该联接类型与ALL相同,除了只有索引树被扫描。这通常比ALL快,因为索引文件通常比数据文件小。
- **** ALL:MySQL将遍历全表以找到匹配的行。
- **** NULL：MySQL在优化过程中分解语句，执行时甚至不用访问表或索引，例如从一个索引列里选取最小值可以通过单独索引查找完成。

     ```sql
     explain select\*from t1 where id = (selectmin(id) from t2);
     ```

possible\_keys **:** 指出MySQL能使用哪个索引在该表中找到行

key : 显示MySQL实际决定使用的键(索引)
key\_len : 显示MySQL决定使用的键长度

ref : 显示使用哪个列或常数与key一起从表中选择行。

Rows **:** 显示MySQL认为它执行查询时必须检查的行数。

**Extra**  **:**

- **** Using filesort:MySQL需要额外的一次传递,以找出如何按排序顺序检索行。

 explain select store\_id from store\_info where id in (1,2) ORDER BY id

 explain select store\_id from store\_info where id in (1,2) ORDER BY store\_id

- **** Using index:从只使用索引树中的信息而不需要进一步搜索读取实际的行来检索表中的列信息。（覆盖索引）

 explain select store\_id from store\_info where store\_id in (1,2) ORDER BY store\_id

- **** Using temporary: 这个值表示使用了内部临时(基于内存的)表。一个查询可能用到多个临时表。有很多原因都会导致MySQL在执行查询期间创建临时表。两个常见的原因是在来自不同表的上使用了DISTINCT,或者使用了不同的ORDER BY和GROUP BY31,1312列。 A,B    b,a

explain select id from store\_info where id in (1,2) GROUP BY store\_id

- **** Using where:WHERE 子句用于限制哪一个行匹配下一个表或发送到客户。
- **** Impossible where
这个值强调了where语句会导致没有符合条件的行。

         EXPLAIN SELECT \* FROM t1 WHERE 1=2;



6.覆盖索引：全值匹配 多列索引 (推荐)

![](mq6.png)
EXPLAIN SELECT \* from store\_info  where store\_id  = 1 and vender\_id =2

如上：用多少个索引？

![](mq7.png)
explain SELECT store\_id  from store\_info where  store\_id = 1 and vender\_id = 1;

explain SELECT store\_id  from store\_info where  store\_id != 1 and  vender\_id = 1;

explain SELECT store\_id  from store\_info where  store\_id != 1;

explain SELECT store\_id  from store\_info where  store\_id = 1 OR vender\_id = 1;

explain SELECT store\_id  from store\_info where  store\_id != 1 OR vender\_id = 1;

explain SELECT store\_id  from store\_info where  store\_id = 1 or vender\_id = 1 ORDER BY store\_id;



7.避免索引失效：（多列索引）

1. 针对多列索引，采用最左匹配法则，中间不能断（包括大约等于 小于）
2. 不能再索引上做任何操作函数和类型转换（下面有讲）
3. 范围查询左侧全失效，  name=&quot;&quot; and age \&gt; 25 and type=&quot;&quot;
4. 尽量使用覆盖索引，用三个字段创建索引，只用其中一两个字段，尽量少用select \* from  user
5. 使用 !=   \&lt;\&gt; 会全表扫描。`

 explain  SELECT \* from store\_label where label\_type\_id  !=1

1. is not null 会全表扫描。

 explain  SELECT \* from store\_label where label\_type\_id  is not null;

1. Like  &quot;%ssss%&quot;  &quot;%sfds&quot;
2. 字符串不加单引号，如：2000  （因为内部使用了 类型转换函数）。
3. 少用or 用它连接查询导致索引失效。

 explain  SELECT \* from store\_label where label\_type\_id  is null  or label\_name like &quot;1123%&quot;;
# 什么是索引？Mysql目前主要的几种索引类型

### 一、索引类型

Mysql目前主要有以下几种索引类型：FULLTEXT，HASH，BTREE，RTREE。

#### 1. FULLTEXT

即为全文索引，目前只有MyISAM引擎支持。其可以在CREATE TABLE ，ALTER TABLE ，CREATE INDEX 使用，不过目前只有 CHAR、VARCHAR ，TEXT 列上可以创建全文索引。

全文索引并不是和MyISAM一起诞生的，它的出现是为了解决WHERE name LIKE “%word%"这类针对文本的模糊查询效率较低的问题。

#### 2. HASH

由于HASH的唯一（几乎100%的唯一）及类似键值对的形式，很适合作为索引。

HASH索引可以一次定位，不需要像树形索引那样逐层查找,因此具有极高的效率。但是，这种高效是有条件的，即只在“=”和“in”条件下高效，对于范围查询、排序及组合索引仍然效率不高。

#### 3. BTREE

BTREE索引就是一种将索引值按一定的算法，存入一个树形的数据结构中（二叉树），每次查询都是从树的入口root开始，依次遍历node，获取leaf。这是**MySQL里默认和最常用的索引类型。**

#### 4. RTREE

RTREE在MySQL很少使用，仅支持geometry数据类型，支持该类型的存储引擎只有MyISAM、BDb、InnoDb、NDb、Archive几种。

相对于BTREE，RTREE的优势在于范围查找。

ps. 此段详细内容见此片博文：Mysql几种索引类型的区别及适用情况

### 二、索引种类

- 普通索引：仅加速查询
- 唯一索引：加速查询 + 列值唯一（可以有null）
- 主键索引：加速查询 + 列值唯一（不可以有null）+ 表中只有一个
- 组合索引：多列值组成一个索引，专门用于组合搜索，其效率大于索引合并
- 全文索引：对文本的内容进行分词，进行搜索



**联合索引**是指对表上的多个列进行索引，联合索引也是一棵B+树，不同的是联合索引的键值数量不是1，而是大于等于2.

**最左匹配原则**

![image-20200919011911398](https://gitee.com/fking86/images4typora/raw/master/imgs/20200919011911.png)

假定上图联合索引的为（a,b）。联合索引也是一棵B+树，不同的是B+树在对索引a排序的基础上，对索引b排序。所以数据按照（1,1),(1,2)......顺序排放。

对于selete * from table where a=XX and b=XX，显然是可以使用(a,b)联合索引的，

对于selete * from table where a=XX，也是可以使用(a,b)联合索引的。因为在这两种情况下，叶子节点中的数据都是有序的。

但是，对于b列的查询，selete * from table where b=XX。则不可以使用这棵B+树索引。可以发现叶子节点的b值为1,2,1,4,1,2。显然不是有序的，因此不能使用(a,b)联合索引。

By the way:selete * from table where b=XX and a=XX,也是可以使用到联合索引的，你可能会有疑问，这条语句并不符合最左匹配原则。这是由于查询优化器的存在，mysql查询优化器会判断纠正这条sql语句该以什么样的顺序执行效率最高，最后才生成真正的执行计划。所以，当然是我们能尽量的利用到索引时的查询顺序效率最高咯，所以mysql查询优化器会最终以这种顺序进行查询执行。

优化：在联合索引中将选择性最高的列放在索引最前面。

例如：在一个公司里以age 和gender为索引，显然age要放在前面，因为性别就两种选择男或女，选择性不如age。



### **三、聚集索引和非聚集索引**

聚集索引存储记录是物理上连续存在，而非聚集索引是逻辑上的连续，物理存储并不连续

字典的拼音查询法就是聚集索引，字典的部首查询就是一个非聚集索引.

聚集索引和非聚集索引的根本区别是表记录的排列顺序和与索引的排列顺序是否一致

聚集索引一个表只能有一个，而非聚集索引一个表可以存在多个。



## 四、建立索引的原则

1) 定义**主键**的数据列一定要建立索引。
2) 定义有**外键**的数据列一定要建立索引。
3) 对于**经常查询**的数据列最好建立索引。
4) 对于需要在**指定范围内的快速或频繁查询**的数据列;
5) **经常用在WHERE子句**中的数据列。

6) 经常出现在关键字**order by、group by、distinct**后面的字段，建立索引。

 如果建立的是复合索引，索引的字段顺序要和这些关键字后面的字段顺序一致，否则索引不会被使用。

7) 对于那些查询中很少涉及的列，重复值比较多的列不要建立索引。

8) 对于定义为text、image和bit的数据类型的列不要建立索引。

9) 对于经常存取的列避免建立索引

10) 限制表上的索引数目。对一个存在大量更新操作的表，所建索引的数目一般不要超过3个，最多不要超过5个。

11) 对复合索引，按照字段在查询条件中出现的频度建立索引。在复合索引中，记录首先按照第一个字段排序。
对于在第一个字段上取值相同的记录，系统再按照第二个字段的取值排序，以此类推。
因此只有复合索引的第一个字段出现在查询条件中，该索引才可能被使用,因此将应用频度高的字段，放置在复合索引的前面，会使系统最大可能地使用此索引，发挥索引的作用。




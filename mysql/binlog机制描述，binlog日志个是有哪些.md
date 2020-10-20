binlog机制描述，binlog日志个是有哪些



# 一、binlog日志介绍

**1）什么是binlog**

binlog是记录所有数据库表结构变更（例如CREATE、ALTER TABLE…）以及表数据修改（INSERT、UPDATE、DELETE…）的二进制日志。

binlog不会记录SELECT和SHOW这类操作，因为这类操作对数据本身并没有修改，但你可以通过查询通用日志来查看MySQL执行过的所有语句。

如果update操作没有造成数据变化，也是会记入binlog。

**2）binlog作用**

因为有了数据更新的binlog，所以可以用于实时备份，与master/slave主从复制结合。

**3）和binlog有关参数**

log_bin

设置此参数表示启用binlog功能，并指定路径名称

log_bin_index

设置此参数是指定二进制索引文件的路径与名称

binlog_do_db

此参数表示只记录指定数据库的二进制日志

binlog_ignore_db

此参数表示不记录指定的数据库的二进制日志

max_binlog_cache_size

此参数表示binlog使用的内存最大的尺寸

binlog_cache_size

此参数表示binlog使用的内存大小，可以通过状态变量binlog_cache_use和binlog_cache_disk_use来帮助测试。

binlog_cache_use：使用二进制日志缓存的事务数量

binlog_cache_disk_use:使用二进制日志缓存但超过binlog_cache_size值并使用临时文件来保存事务中的语句的事务数量

max_binlog_size

Binlog最大值，最大和默认值是1GB，该设置并不能严格控制Binlog的大小，尤其是Binlog比较靠近最大值而又遇到一个比较大事务时，为了保证事务的完整性，不可能做切换日志的动作，只能将该事务的所有SQL都记录进当前日志，直到事务结束

sync_binlog

这个参数直接影响mysql的性能和完整性

sync_binlog=0

当事务提交后，Mysql仅仅是将binlog_cache中的数据写入Binlog文件，但不执行fsync之类的磁盘 同步指令通知文件系统将缓存刷新到磁盘，而让Filesystem自行决定什么时候来做同步，这个是性能最好的。

sync_binlog=n，在进行n次事务提交以后，Mysql将执行一次fsync之类的磁盘同步指令，同志文件系统将Binlog文件缓存刷新到磁盘。

Mysql中默认的设置是sync_binlog=0，即不作任何强制性的磁盘刷新指令，这时性能是最好的，但风险也是最大的。一旦系统绷Crash，在文件系统缓存中的所有Binlog信息都会丢失



# 两个误解：

**1、binlog只是一类记录操作内容的日志文件**

因为binlog称之为二进制日志，很多研发会把这个**二进制日志**和我们平时在代码里写的**代码日志**联系在一起。因为我们的**代码日志**，只有一类记录操作容的文件，并不包含索引文件。然而，这个二进制日志包括两类文件：

- 索引文件（文件名后缀为.index）用于记录哪些日志文件正在被使用
- 日志文件（文件名后缀为.00000*）记录数据库所有的DDL和DML(除了数据查询语句)语句事件。



在文件目录/var/log/mysql/下面发现两个文件mysql-bin.000001和mysql-bin.index。

mysql-bin.index就是我们所说的索引文件，打开瞅瞅，内容是下面这样,记录哪些文件是日志文件。

./mysql-bin.000001

那么说到日志文件。在innodb里其实又可以分为两部分，一部分在缓存中，一部分在磁盘上。这里业内有一个词叫做**刷盘**，就是指将缓存中的日志刷到磁盘上。跟**刷盘**有关的参数有两个个:sync_binlog和binlog_cache_size。这两个参数作用如下

binlog_cache_size: 二进制日志缓存部分的大小，默认值32k

sync_binlog=[N]: 表示写缓冲多少次，刷一次盘,默认值为0

注意两点:

- (1)binlog_cache_size设过大，会造成内存浪费。binlog_cache_size设置过小，会频繁将缓冲日志写入临时文件。
- (2)sync_binlog=0:表示刷新binlog时间点由操作系统自身来决定，操作系统自身会每隔一段时间就会刷新缓存数据到磁盘，这个性能最好。sync_binlog=1，代表每次事务提交时就会刷新binlog到磁盘。sync_binlog=N,代表每N个事务提交会进行一次binlog刷新。

另外，这里存在一个一致性问题，sync_binlog=N，数据库在操作系统宕机的时候，可能数据并没有同步到磁盘，于是再次重启数据库，会带来数据丢失问题。



**2、binlog是InnoDb独有的**

binlog是以事件形式记录的，这句话通俗点说，就是binlog的内容都是一个个的事件。这块具体的我会在下一篇讲，这篇记住binlog的内容就是一个个事件就行。

注意了，这里的用词，是一个个事件，而不是事务。大家应该知道Innodb和mysiam最显著的区别就是一个支持事务，一个不支持事务。

因此你可以说，binlog是基于事务来记录二进制日志，比如sync_binlog=1,每提交一次事务，就写入binlog。你却不能说binlog是事务日志，binlog不仅记录innodb日志，在myisam中，也一样存在binlog。

三个用途

这三个用途，出自《MySQL技术内幕 InnoDB存储引擎》一书，分别为**恢复**、**复制**、**审计**。

**恢复**：如何利用binlog日志恢复数据库数据。就自己去创建个库，然后删了，再去恢复一下数据

**复制**: ![image-20200916142033231](https://gitee.com/fking86/images4typora/raw/master/imgs/20200916142033.png)



主库有一个log dump线程，将binlog传给从库

从库有两个线程，一个I/O线程，一个SQL线程，I/O线程读取主库传过来的binlog内容并写入到relay log,SQL线程从relay log里面读取内容，写入从库的数据库。

**审计**：用户可以通过二进制日志中的信息来进行审计，判断是否有对数据库进行注入攻击。



# 四个常识

**常识一:binlog常见格式**

这块知识我用一个表格来表示，没必要啰嗦一大堆。

![image-20200916142218603](https://gitee.com/fking86/images4typora/raw/master/imgs/20200916142218.png)

业内目前推荐使用的是row模式，准确性高，虽然说文件大，但是现在有SSD和万兆光纤网络，这些磁盘IO和网络IO都是可以接受的。

那么，大家一定想问，为什么不推荐使用mixed模式，理由如下

假设master有两条记录，而slave只有一条记录。

![](https://gitee.com/fking86/images4typora/raw/master/imgs/20200916142234.png)

当在master上更新一条从库不存在的记录时，也就是id=2的记录，你会发现master是可以执行成功的。而slave拿到这个SQL后，也会照常执行，不报任何异常，只是更新操作不影响行数而已。并且你执行命令show slave status，查看输出，你会发现没有异常。但是，如果你是row模式，由于这行根本不存在，是会报1062错误的。

**常识二:怎查看binlog**

binlog本身是一类二进制文件。二进制文件更省空间，写入速度更快，是无法直接打开来查看的。

因此mysql提供了命令mysqlbinlog进行查看。

一般的statement格式的二进制文件，用下面命令就可以

mysqlbinlogmysql-bin.000001

如果是row格式，加上-v或者-vv参数就行，如

mysqlbinlog-vvmysql-bin.000001

**常识三:怎么删binlog**

删binlog的方法很多，有三种是常见的

(1) 使用reset master,该命令将会删除所有日志，并让日志文件重新从000001开始。

(2) 使用命令

PURGE{ BINARY| MASTER} LOGS{ TO'log_name'| BEFOREdatetime_expr }

例如

purgemasterlogsto"binlog_name.00000X"

将会清空00000X之前的所有日志文件.

(3) 使用expire_logs_days=N选项指定过了多少天日志自动过期清空。

**常识四:binlog常见参数**

常见参数，列举如下，有个印象就好。

![image-20200916142303180](https://gitee.com/fking86/images4typora/raw/master/imgs/20200916142303.png)



## 三种工作模式

**1.Statement：每一条会修改数据的sql都会记录在binlog中。**

优点：不需要记录每一行的变化，减少了binlog日志量，节约了IO，提高性能。(相比row能节约多少性能与日志量，这个取决于应用的SQL情况，正常同一条记录修改或者插入row格式所产生的日志量还小于Statement产生的日志量，但是考虑到如果带条件的update操作，以及整表删除，alter表等操作，ROW格式会产生大量日志，因此在考虑是否使用ROW格式日志时应该跟据应用的实际情况，其所产生的日志量会增加多少，以及带来的IO性能问题。)

缺点：由于记录的只是执行语句，为了这些语句能在slave上正确运行，因此还必须记录每条语句在执行的时候的一些相关信息，以保证所有语句能在slave得到和在master端执行时候相同的结果。另外mysql 的复制,像一些特定函数功能，slave可与master上要保持一致会有很多相关问题(如sleep()函数， last_insert_id()，以及user-defined functions(udf)会出现问题).

**2.Row:不记录sql语句上下文相关信息，仅保存哪条记录被修改。**

优点： binlog中可以不记录执行的sql语句的上下文相关的信息，仅需要记录那一条记录被修改成什么了。所以rowlevel的日志内容会非常清楚的记录下每一行数据修改的细节。而且不会出现某些特定情况下的存储过程，或function，以及trigger的调用和触发无法被正确复制的问题

缺点:所有的执行的语句当记录到日志中的时候，都将以每行记录的修改来记录，这样可能会产生大量的日志内容,比如一条update语句，修改多条记录，则binlog中每一条修改都会有记录，这样造成binlog日志量会很大，特别是当执行alter table之类的语句的时候，由于表结构修改，每条记录都发生改变，那么该表每一条记录都会记录到日志中。

**3.Mixedlevel: 是以上两种level的混合使用**，一般的语句修改使用statment格式保存binlog，如一些函数，statement无法完成主从复制的操作，则采用row格式保存binlog,MySQL会根据执行的每一条具体的sql语句来区分对待记录的日志形式，也就是在Statement和Row之间选择一种.新版本的MySQL中队row level模式也被做了优化，并不是所有的修改都会以row level来记录，像遇到表结构变更的时候就会以statement模式来记录。至于update或者delete等修改数据的语句，还是会记录所有行的变更。
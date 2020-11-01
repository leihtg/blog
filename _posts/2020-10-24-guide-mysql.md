---
layout: post
title: 数据库知识点
categories: guide
tags: guide mysql database
author: leihtg
---

* content
{:toc}



### 数据库

事务的四大特征    
1. 原子性 （atomicity）:强调事务的不可分割.  
2. 一致性 （consistency）:事务的执行的前后数据的完整性保持一致.
3. 隔离性 （isolation）:一个事务执行的过程中,不应该受到其他事务的干扰
4. 持久性（durability） :事务一旦结束,数据就持久到数据库
#### 性能优化
1. sql优化
2. 索引的使用

#### mysql
1. 问的聚簇索引、什么是幻读、b+树为什么快

密集索引和稀疏索引的区别
* 密集索引文件中的每个搜索码值都对应一个索引值
* 稀疏索引文件只为码的某些值建立索引项

InnoDB
* 若一个主键被定义，该主键则作为密集索引

* 若没有主键被定义，该表的第一个唯一非空索引则作为密集索引

* 若不满足以上条件，innodb内部会生成一个隐藏主键（密集索引）

* 非主键索引存储相关键位和其对应的主键值，包含两次查找

  

InnoDB有且仅有一个聚集索引，索引是和数据文件绑在一起的，必须要有主键。通过主键索引效率很高，但是辅助索引需要查两次，先查到主键然后再通过主键查询到数据。而MyISAM是非聚集索引，数据文件和索引文件是分离的，索引是保存数据文件的指针，而主键索引和辅助索引是独立的，因此MyISAM在纯检索系统中，也就是增删改很少的系统中，其性能要好于InnoDB。

MyISAM适合的场景

1. 频繁执行全表count语句
2. 对数据进行增删改的频率不高，查询非常频繁
3. 没有事务

InnoDB适合的场景

1. 数据增删改查都相当频繁
2. 可靠性要求比较高，要求支持事务



关于索引的问题：

- 为什么要使用索引
  避免全表扫描,提升检索效率
  
- 什么样的信息能成为索引
  主键、唯一键等，只要能让数据具备一定区分性的字段
  
- 索引的数据结构  

   主流是B+-Tree，还有hash，bitmap等，其中mysql不支持bitmap

  

衍生出来的问题，以mysql为例

* 如何定位并优化慢查询sql

  1. 根据慢日志定位慢查询sql

     ```sql
     SHOW VARIABLES LIKE '%query%';
     SHOW STATUS LIKE '%slow_qu%';
     
     SET GLOBAL slow_query_log = ON;
     SET GLOBAL long_query_time = 1;
     ```

     

  2. 使用explain等工具分析sql

     explain中的type为 index或者all, 证明语句是需要优化.

     extra: using filesort, using temporary,表示也需要优化

  3. 修改sql或者尽量让sql走索引

     ```sql
     select count(id) from table force index(primary) ;#强制使用主键索引
     ```

     

* 联合索引的最左匹配原则的成因

  1. 最左前缀匹配原则，是非常重要的原则，mysql会一直向右匹配直到遇到范围查询（>,<,between,like）就停止匹配，比如`a=3 and b=4 and c>5 and d=6`如果建立（a,b,c,d)顺序的索引，d是用不到索引的，如果建立（a,b,d,c）的索引则都可以用到。a,b,d的顺序可以任意调整

  2. =和in可以乱序，比如 `a=1 and b=2 and c=3` 建立（a,b,c）索引可以任意顺序，mysql的查询优化器会帮你优化成索引可以识别的形式。

  我们的回答最左匹配原则的成因：mysql创建复合索引的规则呢是首先会对复合索引的最左边，那也就是第一个所以字段的数据进行排序。在第一个字段的排序的基础上呢再对后面第二个，索引字段进行排序，其实就相当于实现了类型order by字段一。在order by 字段二这样一种排序规则，那么所以呢，第一个字段是绝对有序的，而第二个字段就是无序的了，因此通常情况下直接使用第二个字段进行条件判断是用不到索引的，这就是所谓的为什么要强调联合索引这种匹配原则的原因。咱们来看他们一个例子。

  

* 索引是建立的越多越好吗

  1. 数据量小的表不需要建立索引，建立会增加额外的索引开销
  2. 数据变更需要维护索引，因此更多的索引意味着更多的维护成本
  3. 更多的索引意味着也需要更多的空间



#### 锁模块

##### 数据库锁的分类

1. 按锁的粒度划分，可分为表级锁、行级锁、页级锁(数据块)
2. 按锁级别划分，可分为共享锁、排他锁
3. 按加锁方式划分，可分为自动锁、显示锁
4. 按操作划分，可分为DML锁、DDL锁
5. 按使用方式划分，可分为乐观锁、悲观锁

##### 常见问题

1. MyISAM与InnoDB关于锁方面的区别是什么

   * MyISAM默认用的是表级锁，不支持行级锁
   * InnoDB默认用的是行级锁，也支持表级锁

   加读/写锁

   ```sql
   select * from xxx for update; -- 加排他锁
   
   select * from xxx lock in share mode;--加共享锁
   
   lock tables xxx read/write;
   
   unlocks tables;
   ```

   

   事务默认自动提交

   ```sql
   show variables like 'autocommit';
   
   set autocommit=0; --关闭自动提交
   ```

   

   InnoDB在sql没有用到索引的时候用的是表级锁，在sql用到索引的时候用的是行级锁及间隙锁（普通非惟一索引）

   InnoDB还支持表级的意向锁，意向锁也分为共享读锁（IS），排他写锁(IX)

   

2. 数据库事务的四大特性（ACID）

   1. 原子性（Atomic）
   2. 一致性（Consistency）
   3. 隔离性（Isolation）
   4. 持久性（Durability）

   

3. 事务隔离级别以及各级别下的并发访问问题

   1. 更新丢失---mysql所有事务隔离级别在数据库层面上均可避免

   2. 脏读--READ-COMMITTED事务隔离级别以上可避免

      ```sql
      select @@tx_isolation; -- 查看事务隔离级别
      set session transaction isolation level read uncommitted; -- 设置为读未提交
      start transaction; --开启事务
      commit;
      ```

   3. 不可重复读——REPEATABLE-READ事务隔离级别以上可避免

   4. 幻读——SERIALIZABLE事务隔离级别可避免

      ```sql
      select * from xxx lock in share mode; -- 也叫当前读
      ```

      

4. InnoDB可重复读隔离级别下如何避免幻读

   1. 表象： 快照读（非阻塞读）——伪MVCC

   2. 内存： next-key锁（行锁+gap锁）

      RC/read-uncommit以及更低事务隔离级别下没有gap锁，所以是避免不了幻读的原因。RR,SERALIZABLE级别下默认都支持Gap锁。

      对主键索引或唯一索引会用gap锁吗

      1. 如果where条件全部命中，则不会用Gap锁，只会加记录锁
      2. 如果where条件部分命中或者全不命中，则会加Gap锁

      Gap锁会用在非唯一索引或者不走索引的当前读中

      1. 非唯一索引
      2. 不走索引（会对所有gap都加锁）

   当前读和快照读

   * 当前读： `select .. lock in share mode;`  `select .. fro update;`

   * 当前读：`update, delete, insert`

     当前读是加了锁的增删改查语句，不管是共享锁、排他锁均为当前读，为什么叫当前读呢，因为它读取的是记录的最新版本。并且保证读取之后还需要保证其他并发事务不能修改当前记录，对读取的记录加锁。

     ![]({{site.baseurl}}/assets/20201024/微信截图_20201024203720.png)

   * 快照读： 不加锁的非阻塞读， select

5. RC、RR级别下的InnoDB的非阻塞读如何实现

   1. 数据行里的DB_TRX_ID(事务id)、DB_ROLL_PTR(回滚指针)、DB_ROW_ID字段

   2. undo日志

      ![]({{site.baseurl}}/assets/20201024/微信截图_20201024205138.png)

      ![]({{site.baseurl}}/assets/20201024/微信截图_20201024205305.png)

      

   3. read view 

      可见性判断

![]({{site.baseurl}}/assets/20201024/微信截图_20201024194623.png)

##### 常见问题

1. MyISAM与InnoDB关于锁方面的区别是什么
2. 数据库事务的四大特性
3. 事务隔离级别以及各级别下的并发访问问题
4. InnoDB可重复读隔离级别下如何避免幻读
5. RC、RR级别下的InnoDB的非阻塞读如何实现

#### 语法部分

1. ##### group by 

   满足“select子句中的列名必须为分组列或列函数

   列函数对于group by 子句定义的每个组各返回一个结果

2. ##### having

   通常与group by 子名一起使用

   where过滤行，having过滤组

   出现在同一sql的顺序： where > group by > having

   如果没有group ， having相当于where

   ```sql
   select * from stu having id=1;
   ```

   

3. ##### 统计相关：count, sum, max, min, avg
ount, sum, max, min, avg
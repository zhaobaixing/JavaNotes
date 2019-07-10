
<!-- GFM-TOC -->
* [一、存储引擎](#存储引擎)
* [二、事务](#事务)
    * [什么是事务](##什么是事务？他有什么特性？)
    * [事务隔离](##事务隔离)
* [三、锁机制](#锁机制)
* [四、索引](#索引)
* [五、分库分表](#分库分表)
    * [水平拆分](##水平拆分)
    * [垂直拆分](##垂直拆分)
<!-- GFM-TOC -->

# 存储引擎
MyISAM作为mysql5.5之前的默认存储引擎拥有很多性能优势。比如全文索引、压缩等。MyISAM只支持表锁，不支持事务，崩溃后表数据无法恢复。
InnoDB作为5.5版本后的默认引擎。锁粒度包含表锁行锁。由redo log,undo log支持的事务。崩溃后表数据可以恢复。

我们把MyISAM和InnoDB做一个对比：

1.<font color=red>是否支持行锁</font>：
MyISAM只支持表锁，而InnoDB支持表锁和行锁，默认行锁，在无索引无主键情况更新才会锁表。

2.<font color=red>是否支持事务和崩溃恢复</font>：
MyISAM作为性能查询表不支持事务和宕机恢复，而InnoDB则支持事务提交和宕机恢复。

3.<font color=red>是否支持mvcc:仅InnoDB支持mvcc:</font>
mvcc仅在读已提交和可重复读两个隔离状态下工作。因为读未提交会出现脏读情况所以不适合mvcc，而串行化隔离状态下的表加锁非行级锁所以用不到mvcc的行级锁
| 引擎 | 行锁 | 事务 | mvcc|
| ------ | ------ | ------ | ------ |
| MyISAM | 否 | 否 | 否 |
| InnoDB |是 | 是 |是 |

# 事务
### 什么是事务？他有什么特性？
<font color=red>指作为单个逻辑工作单元执行的一系列操作，要么完全地执行，要么完全地不执行。</font>

提到事务首先想到事务的<font color=red>ACID</font>
事务必须满足的ACID为

1.<font color=red>原子性Atomicity</font>：
原子性就包含对事务的定义。即一个事务是执行的最小单元，要么都提交，要么都回滚。

2.<font color=red>一致性Consistency</font>：
事务执行前后的数据必须是完整的。

3.<font color=red>隔离性Isolation：</font>
当数据库并发访问时，数据库必须对事务做出一个隔离级别规范。引入四大事务隔离的概念。读未提交，读已提交，可重复度，串行化。

4.<font color=red>持久性Durability：</font>
当事务提交后，其对数据的修改即为永久的。就算数据库崩溃故障也不应对数据有任何影响。

### 事务隔离
由于数据库并发事务的存在。由于不同的事务隔离等级导致的，脏读、不可重复读、幻读。

1.脏读为一个事务读取到了另个事务未提交的数据，这种问题通常发生在读未提交(Read UnCommitted)的事务隔离级别
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619233909709.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tlZXBWaXNpb24=,size_16,color_FFFFFF,t_70)
2.不可重复读通常发生在读已提交级别下的所有级别（Read Committed）包括读未提交，不可重复读强调更新。
当事务A开启并第一次查询t_user表数据这时候我们看到的是原始数据，name叫小王，随后事务B更新name为balabala并提交事务B,
这时A事务再次查询时则name已经是balabala而不是小王了。这就是不可重复读。
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019061923564097.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tlZXBWaXNpb24=,size_16,color_FFFFFF,t_70)
3.幻读，幻读通常发生在可重复读隔离级别下的所有级别，包括读未提交、读已提交级别。幻读强调插入和删除。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190620000708393.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tlZXBWaXNpb24=,size_16,color_FFFFFF,t_70)
四种隔离级别强度:读未提交<读已提交<可重复读<串行化
值得一提的是mysql默认的隔离机制为可重复读，隔离机制越严格数据库的性能也会越低，更容易发生死锁。

# 锁机制
MyIsAM仅使用表级锁（table-level-lock），InnoDB除了表锁还支持行级锁,默认使用行级锁（row-level-lock）

行锁的三种算法
- Record Lock:单个行记录上的锁

- Gap Lock:间隙锁,锁定一个范围，但不包含记录本身

- Next-key Lock:Gap Lock+Record Lock锁定一个范围，并且锁定记录本身

Record Lock:在使用主键索引或者唯一索引作查询条件更新的话则使用Record Lock锁住单条记录更新。

Gap Lock为间隙锁。是针对于索引区间的锁。mysql表内的锁都是针对索引，间隙锁不包括查询的记录本身即（索引1，索引2）,有两种情况可以显示关闭Gap Lock;1.将事务的隔离级别设成RC。2.将参数innodb_locks_unsafe_for_binlog 设置为1

Next-key Lock：为Gap Lock+Record Lock 在mysql默认事务隔离级别内开启，为了解决幻读（但是必须建立在有索引条件插入的情况下），Next-key-Lock区间为，(索引1，索引2]

# 索引
mysql索引使用的是B+树
联合索引情况下mysql默认最左原则。即如果有一个联合索引name,sex,age.我们要查找（小赵,18,F）的mysql会先比较name再以此比对age,sex进而搜索出结果集。如果查询条件为（18，F），mysql索引无法找到比对的name，则无法利用这个联合索引进而全表扫描。

# 分库分表
### 水平拆分 
水平拆分是指数据表行的拆分，表的行数超过200万行时，就会变慢，这时可以把一张的表的数据拆成多张表来存放。 优点就是客户端基本不需要改动。其缺点就是要解决数据分片。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190623202407988.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tlZXBWaXNpb24=,size_16,color_FFFFFF,t_70)

### 垂直拆分 
垂直拆分是把原本数据量较大，列较多的表拆分成多张表。分散单表索引量，提升单表查询效率。其缺点是数据表难以维护，表间关系复杂外键过多。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190623202511507.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tlZXBWaXNpb24=,size_16,color_FFFFFF,t_70)

<font color=red>事务分片的几种解决方案：</font>
- <font color=red>客户端代理jar包：当当网的sharding-jdbc(开源)</font>，阿里的TDDL（暂未开源）
-  <font color=red>中间件代理：在应用和数据见加了一个代理层，分配逻辑统一在中间件服务中处理</font>Mycat,Atlas,DDB等
<!-- GFM-TOC -->
* [一、redis原理](#redis原理)
* [二、reidis数据结构](#reidis数据结构)
* [三、reidis集群、主从机制](#reidis集群、主从机制)
* [四、redis哨兵-Sentinel ](#redis哨兵-Sentinel)
* [五、redis订阅发布](#redis订阅发布)
* [六、redis内存淘汰机制](#redis内存淘汰机制)
* [七、使用场景](#使用场景)
* [八、nosql的比较](#nosql的比较)
<!-- GFM-TOC -->

# redis原理
redis的读写是采用了<font color=red>**单线程的 io 多路复用**</font>，那么什么是io多路复用技术？什么是io。
我们先引入一个io的流程
- 应用程序请求获取数据，用户态请求内核读取数据，然后等待内核读取数据。
- 内核把read的数据复制到用户空间。然后才能使用数据。

因为read是内核才能完成的事，用户线程在请求内核线程的时候有可能会挂起阻塞。就有了3大io模型<font color=red> **BIO,NIO,AIO,多路复用**</font>

1）**BIO**：blocking io 为同步阻塞 IO
- 应用请求数据，请求内核资源，BIO会挂起用户线程，直至内核资源空闲或数据复制完毕返回给用户线程
- 这个过程为同步阻塞
- 一个线程负责一个io读写，在并发读写情况下非常的消耗线程资源。而计算机的线程是固定的，所以在大量io会造成系统崩溃
![在这里插入图片描述](/assects/database/BIO.png)
2）**NIO**：New io为系统资源的使用提供了一套较为有效的办法，自旋和selector复用线程。
- NIO的核心思想就是多路复用。
- NIO通过Selector构建多路复用器，使用一个或少量线程监听大量socket链接。
- 通过select,poll,epoll等机制处理用户线程
![在这里插入图片描述](/assects/database/NIO.png)
3)**AIO**：异步非阻塞型io,用户链接无需等待响应，响应由回调函数通知完成
- 用户线程异步非阻塞
![在这里插入图片描述](/assects/database/AIO.jpg)

**阻塞或非阻塞**：是否需要等待资源使用。当资源没准备好而挂起线程则为阻塞，资源没准备好返回状态可以重试等操作的即为非阻塞

**同步或者异步**：是否需要等待结果集返回的，如果需要等待结果返回则为同步，如果不需要等待结果的则为异步，一般异步有回调方法，成功后由os通知用户结果。

# reidis数据结构
redis数据结构为k-v对应的数据，不同于mysql,oracle等的数据行

## String

```
>set key value
OK
>get key
value
>del key
1
```

## List

### 有序列表，实现为链表
```
---添加集合元素
>lpush testKey value1
1
>lpush testKey value2
1
---获取所有集合元素
>lrange testKey 0 -1
1) value2
2) value1
---从链表头部拿出一个元素
>lpop testKey
value2
```

## Set

### 无序列表，通过hash实现插入，不允许重复，查询复杂度 O(1)

```
---添加集合元素
>sadd testKey value1
1
>sadd testKey value2
1
---获取集合成员数
>SCARD testKey
2
---获取集合成员
>smembers testKey
value2
value1
---从set中拿出一个元素
>spop testKey
value2
```

## Zset

### 有序列表

```
---添加集合元素
>zadd testKey 1 value1
1
>zadd testKey 2 value2
1
---获取集合成员数
>zcard testKey
2
---获取集合成员
>zrange testKey 0 -1
value1
value2
---从zset中拿出一个元素
>zrem testKey value2
value2
```

# reidis集群、主从机制
 redis 提供了集群机制应对单机瓶颈，还有不错主从的容错机制。

# redis哨兵-Sentinel
当集群情况下，哨兵担任监听者的职责观察，当主服务 down 掉会在从服务器中推举出新的主服务器。

# redis订阅发布
redis还提供发布订阅功能，思想相当于观察者模式

# redis内存淘汰机制

| 策略名 | 描述 |
| ------ | ------ |
| volatile-lru |从已设置过期时间的数据集中挑选最近最少使用的数据淘汰|
| volatile-ttl |从已设置过期时间的数据集中挑选将要过期的数据淘汰|
| volatile-random |从已设置过期时间的数据集中任意选择数据淘汰|
| allkeys-lru |从所有数据集中挑选最近最少使用的数据淘汰|
| allkeys-random |从所有数据集中任意选择数据进行淘汰|
| noeviction |禁止驱逐数据|

# 使用场景

### 缓存
redis作为k-v的非关系型数据库，数据存储在内存当中。相对于关系型数据库的普遍存储方式来说，io 读写非常快。

### 消息队列
redis存在list,set等集合。可以使用线程监听等方法pop出栈消费消息，push入栈消息。一般最好选用kafka,rabbitmq等可靠的消息中间件做消息队列。

# nosql的比较

Redis、Memcache和MongoDB的对比

| nosql | 性能 | 灾备回复 |
| ------ | ------ | ------ |
| Redis |高|支持持久化操作（快照、AOF）|
| Memcache |高|不支持|
| MongoDB |高|支持持久化操作（binlog）|
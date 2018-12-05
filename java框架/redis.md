---
title: 2018-12-05redis
tags: redis
grammar_cjkRuby: true
---
# redis

Redis是一个开源的使用ANSI C语言编写、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API。从2010年3月15日起，Redis的开发工作由VMware主持。

作者，叫Salvatore Sanfilippo。

### 1.定义

redis是一个key-value存储系统。

和Memcached类似，它支持存储的value类型相对更多，包括string(字符串)、list(链表)、set(集合)、zset(sorted set --有序集合)和hash（哈希类型）。这些数据类型都支持push/pop、add/remove及取交集并集和差集及更丰富的操作，而且这些操作都是原子性的。在此基础上，redis支持各种不同方式的排序。与memcached一样，为了保证效率，数据都是缓存在内存中。区别的是redis会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件，并且在此基础上实现了master-slave(主从)同步。

Redis 是一个高性能的key-value数据库。 redis的出现，很大程度补偿了memcached这类key/value存储的不足，在部分场合可以对关系数据库起到很好的补充作用。它提供了Java，C/C++，C#，PHP，JavaScript，Perl，Object-C，Python，Ruby，Erlang等客户端，使用很方便。

Redis支持主从同步。数据可以从主服务器向任意数量的从服务器上同步，从服务器可以是关联其他从服务器的主服务器。这使得Redis可执行单层树复制。存盘可以有意无意的对数据进行写操作。由于完全实现了发布/订阅机制，使得从数据库在任何地方同步树时，可订阅一个频道并接收主服务器完整的消息发布记录。同步对读取操作的可扩展性和数据冗余很有帮助。

### 2.三级缓存架构

![缓存架构](\asserts\缓存架构.png)



#### 2.1	Nginx层

对于传统的缓存系统，请求到达nginx，然后分发到对应的服务系统，然后查询redis中是否有数据，然后将结果返回，这里会花费很大的网络开销。nginx可以支持热数据的缓存，对这些数据直接缓存在nginx内存中，请求到来直接返回，不需要去发送网络请求。但是nginx内存的大小一般会很小，所以适合缓存热数据。

#### 2.2	redis cluster层 

这一层的数据是最完整的，大部分的离散请求都会访问到这一层。因此一定要做到高可用，可水平伸缩的模式。

#### 2.3	tomcat层

tomcat中的堆也可以缓存一定的数据，但是容量也不会太大。主要是用来防止redis层大面积崩溃之后，大量数据请求直接到DB层。

#### 2.4	修改数据（解决方案）

对于数据不同的实时性要求，我们可以采取不同的方式来修改缓存

##### 2.4.1对于实时性较高的数据（例如库存数据）

对于这种实时性很高的数据，当有修改的时候，一般是修改DB然后同时去修改redis缓存的双写模式。但是这里会设计到数据不一致的问题。

**1.最初级不一致问题及解决方案** 

对于先修改数据库，再删除缓存，如果缓存删除失败，那么回导致数据库中是最新数据，缓存中是旧数据，出现不一致的情况。 

解决方法：先删除缓存，再修改数据库。

**2.比较复杂的数据不一致问题** 

对于高并发的请求，读写操作可能会产生不一致的问题，在写的过程中，另一个读操作重新读取到来旧数据。当然对于并发不高的系统，这种情况也基本不会遇到。 

解决方案如下：

数据库、缓存更新与读取操作进行异步串行化。

将需要更新的数据，发送到jvm内部的队列中，一般会维护多个内存队列，对数据的唯一标识hash然后对内存队列数量取模，就可以保证同样的数据操作一定会在同一个队列中去。

每个队列有一个工作线程，每个工作线程串行拿到对应的操作，然后逐条执行。因此一个数据的变更，会先删除缓存，然后再去更新数据库，当还没有更新完成的时候，另一个读请求过来来，读到空的缓存，那么可以将这个缓存更新的请求发送到队列中，从而会同步等待缓存更新完成。需要注意的一点，对于缓存为空，多个读请求过来，将多个更新缓存的请求发送到队列是没有必要的，因此需要做过滤，然后可以将这些读请求hang住一个时间段，轮询去查询缓存，当前面队列中的更新缓存请求执行完成后，就可以拿到缓存数据直接返回，否则，可以让服务直接取查询DB来拿数据。

在高并发下，需要注意：

1.对于缓存中没有该数据，也可能是数据库本身就没有。因此需要判断队列中是否有更新该数据的操作，如果没有则读请求没有必要hang住，直接返回空即可。

2.需要线上模拟压力测试，当内存队列积压过多的更新数据操作，会对最后的读操作产生于一定的延时，此时如果需要优化就需要加机器，分配到更平均的队列数量中。

基于zookeeper分布式锁解决多服务缓存重建并发冲突：

对于多个缓存服务，可能存在这种情况，nginx请求到达发现都没有相应缓存数据，会发送请求给缓存服务到数据源拉取数据并写入相应tomcat/redis，同时kafka生产了消息的变更记录也需要去拉取数据并存入tomcat/redis。两者拉取数据源与缓存重建可能会出现并发冲突导致tomcat/redis中的最终数据并不是最新。

##### 2.4.2对于实时性不高的数据

对于实时性不高的数据，如果发生了变更，几分钟之后才更新到页面上，我们采取异步更新缓存的策略。缓存数据生产服务，监听一个消息队列，然后数据源服务（商品信息管理服务）发生来变更之后，就将数据变更的消息推送到消息队列（topic），缓存数据生产服务可以去消费这些变更的信息，然后根据消息的提示提取一些参数，然后调用对应的数据源服务接口拉取数据，一般是从mysql拉取，然后将数据存放到本地堆缓存和redis缓存。

##### 2.4.3对于nginx层的缓存

一般来说，默认会部署多个nginx，在里面都会放一些缓存，此时缓存的命中率会非常底，因为对同样数据的请求可能被路由到多个nginx从而未能找到相应缓存而多次对redis发起请求。因此，可以采用分发层+应用层双层ngin来提升缓存命中率。

分发层nginx：负责流量分发的逻辑和策略，根据自己定义的一些规则，比如根据productId进行hash，然后对后端nginx数量取模将某一个商品的访问请求固定路由到一个nginx后端服务器上去，保证了nginx只会对该商品数据从redis获取一次。之后的同样商品请求都直接走nginx缓存。

后端nginx： 称之为应用服务器。

### 3.redis和memcached、mongoDB对比

Redis 只会缓存所有的key的信息，如果Redis发现内存的使用量超过了某一个阀值，将触发swap的操作，Redis根据“swappability = age*log(size_in_memory)”计算出哪些key对应的value需要swap到磁盘。然后再将这些key对应的value持久化到磁 盘中，同时在内存中清除。这种特性使得Redis可以保持超过其机器本身内存大小的数据。当然，机器本身的内存必须要能够保持所有的key，毕竟这些数据 是不会进行swap操作的。

同时由于Redis将内存中的数据swap到磁盘中的时候，提供服务的主线程和进行swap操作的子线程会共享这部分内存，所以如果更新需要swap的数据，Redis将阻塞这个操作，直到子线程完成swap操作后才可以进行修改。

当 从Redis中读取数据的时候，如果读取的key对应的value不在内存中，那么Redis就需要从swap文件中加载相应数据，然后再返回给请求方。 这里就存在一个I/O线程池的问题。在默认的情况下，Redis会出现阻塞，即完成所有的swap文件加载后才会相应。这种策略在客户端的数量较小，进行 批量操作的时候比较合适。但是如果将Redis应用在一个大型的网站应用程序中，这显然是无法满足大并发的情况的。所以Redis运行我们设置I/O线程 池的大小，对需要从swap文件中加载相应数据的读取请求进行并发操作，减少阻塞的时间。

**redis、memcache、mongoDB 对比**

从以下几个维度，对redis、memcache、mongoDB 做了对比：

1、**性能**

都比较高，性能对我们来说应该都不是瓶颈。

总体来讲，TPS方面redis和memcache差不多，要大于mongodb。

2、**操作的便利性**

memcache数据结构单一。

redis丰富一些。

数据操作方面，redis更好一些，较少的网络IO次数，mongodb支持丰富的数据表达，索引，最类似关系型数据库，支持的查询语言非常丰富。

3.**内存空间的大小和数据量的大小**

redis在2.0版本后增加了自己的VM特性，突破物理内存的限制；可以对key value设置过期时间（类似memcache）。

memcache可以修改最大可用内存,采用LRU算法。

mongoDB适合大数据量的存储，依赖操作系统VM做内存管理，吃内存也比较厉害，服务不要和别的服务在一起。

  4、**可用性（单点问题）**

对于单点问题，

redis，依赖客户端来实现分布式读写；主从复制时，每次从节点重新连接主节点都要依赖整个快照,无增量复制，因性能和效率问题，所以单点问题比较复杂；不支持自动sharding,需要依赖程序设定一致hash 机制。
一种替代方案是，不用redis本身的复制机制，采用自己做主动复制（多份存储），或者改成增量复制的方式（需要自己实现），一致性问题和性能的权衡。

Memcache本身没有数据冗余机制，也没必要；对于故障预防，采用依赖成熟的hash或者环状的算法，解决单点故障引起的抖动问题。

mongoDB支持master-slave,replicaset（内部采用paxos选举算法，自动故障恢复）,auto sharding机制，对客户端屏蔽了故障转移和切分机制。  

  5、**可靠性（持久化）**

对于数据持久化和数据恢复，

redis支持（快照、AOF）：依赖快照进行持久化，aof增强了可靠性的同时，对性能有所影响

memcache不支持，通常用在做缓存,提升性能；

MongoDB从1.8版本开始采用binlog方式支持持久化的可靠性  

  6、**数据一致性（事务支持）**

Memcache 在并发场景下，用cas保证一致性

redis事务支持比较弱，只能保证事务中的每个操作连续执行

mongoDB不支持事务  

  7、**数据分析**

mongoDB内置了数据分析的功能(mapreduce),其他不支持  

  8、**应用场景**

redis：数据量较小的更性能操作和运算上

memcache：用于在动态系统中减少数据库负载，提升性能;做缓存，提高性能（适合读多写少，对于数据量比较大，可以采用sharding）

MongoDB:主要解决海量数据的访问效率问题  

### 4.redis中的常见数据类型及操作

```
通用操作：

  有3个通配符 *, ? ,[]
  *: 通配任意多个字符
  ?: 通配单个字符
  []: 通配括号内的某1个字符

  keys *： 查询所有的key
  keys o*：查询以o开头的key
  keys *o：查询以o结尾的key
  keys ???：查询是3个字符的key
  keys on?:查询是三个字符，前面2个是on的
  keys on[eaw]：查询是以on开始后面是eaw中的任意字符
  
  type key 去获得这个key的数据结构类型

  判断一个键是否存在
  exists key
  删除键
  del key
  del key1 key2

   ttl key
  ttl  site： 查询site这个key的有效期（秒数）
  返回-1 ：永久有效
  返回-2：不存在的key（Redis2.8中对于不存在的key,返回-2）

  expire key 整型值
  作用: 设置key的生命周期,以秒为单位

  同理: 
  pexpire key 毫秒数, 设置生命周期
  pttl  key, 以毫秒返回生命周期

  persist key
  作用: 把指定key置为永久有效
```



#### 4.1	String（字符串）

String是简单的 key-value 键值对，value 不仅可以是 String，也可以是数字。String在redis内部存储默认就是一个字符串，被redisObject所引用，当遇到incr,decr等操作时会转成数值型进行计算，此时redisObject的encoding字段为int。

**相关命令**

```
    Set key value
    get key

    mset key1 value1 key2 value2
    mget key1 key2
    set如果key存在则会覆盖之前的值，如果使用setnx则不会

    setnx key value
    incr x:每次对x对应的值加1
    incrby x 3：每次加3
    incrbyfloat x 2.1：每次加2.1 ，可以指定小数，但是没有decrbyfloat
    decr x：每次对x对应的值减1
    dcr x 3：每次减3
```



#### 4.2	List（列表）

Redis列表是简单的字符串列表，可以类比到C++中的std::list，简单的说就是一个链表或者说是一个队列。可以从头部或尾部向Redis列表添加元素。列表的最大长度为2^32 - 1，也即每个列表支持超过40亿个元素。

Redis list的实现为一个双向链表，即可以支持反向查找和遍历，更方便操作，不过带来了部分额外的内存开销，Redis内部的很多实现，包括发送缓冲队列等也都是用的这个数据结构。

```
    list操作：
    List是有序的字符串列表，是使用双向链表实现
    lpush list a ：向左边添加
    rpush list b ：从右边添加
    lpop list    :从左侧弹出一个数据
    rpop list   :从右侧弹出一个数据
    llen list     ：查看有多少元素
    lrange list 0 3  ：从list中取出从0-3个元素
    lrange list 0 -1  ：从list中取出从0到倒数第一个元素（也就是list中所有的数据）
    lrem list 2 a   :从左边开始找，找2个a并且从list中删除
    lrem list -1 a  ：表示从右侧开始找　找一个ａ并删除
    lrem list 0 a   :表示从list中把所有的a都删除
    lindex list 1  ：查询list中下标为1的数据
    lset list 0 a :将list中下标为0的值改为a
    Ltrim list 0 2  ：截取list中0到2的字符
    Rpoplpush list list1 ：从list中rpop，pop的值lpush到list1
```



#### 4.3	Hash（字典）

类似C#中的dict类型或者C++中的hash_map类型。

Redis Hash对应Value内部实际就是一个HashMap，实际这里会有2种不同实现，这个Hash的成员比较少时Redis为了节省内存会采用类似一维数组的方式来紧凑存储，而不会采用真正的HashMap结构，对应的value redisObject的encoding为zipmap,当成员数量增大时会自动转成真正的HashMap,此时encoding为ht。

```
    hash操作：
    hset user:100 name zs:key为user:100
    hget user:100 name  :获取key为user:100的name
    hmset user:100 name zs age 20
    hmget user:100 name age
    hexists user:100 name  ：查询user:100的name是否存在
    hincrby user:100 age 5  :将user:100的age加5
    hdel user:100 name  :删除user:100的name
    hkeys user:100   :列出user:100的所有字段
    hvals user:100  :列出user:100的所有值
    hlen user:100  :获取user:100包含多少属性
```



#### 4.4	Set（集合）

可以理解为一堆值不重复的列表，类似数学领域中的集合概念，且Redis也提供了针对集合的求交集、并集、差集等操作。

set 的内部实现是一个 value永远为null的HashMap，实际就是通过计算hash的方式来快速排重的，这也是set能提供判断一个成员是否在集合内的原因。

```
    set操作：
    无序不重复
    sadd set a: 添加一个元素
    smembers set :显示出set中所有元素
    srem set a b :从set中删除 a b
    sismember set a :看set中是否存在a元素
    sdiff set set1  :把第一个集合中没有的元素返回出来
    sinter set set1  :交集
    sunion set set1 ：不重复的都列出来
    sunionstore aa set set1 :将set和set1求并集编程在aa集合中
    同理有sdiffstore,sinterstore
    spop set :从set中取出一个元素并删除
    Srandmember set 2  :从set中随机取出2个元素并且不重复
    Srandmember set  -2  :从set中随机取出2个元素但是不保证不重复
```



#### 4.5	Sorted Set（有序集合）

Redis有序集合类似Redis集合，不同的是增加了一个功能，即集合是有序的。一个有序集合的每个成员带有分数，用于进行排序。

Redis有序集合添加、删除和测试的时间复杂度均为O(1)(固定时间，无论里面包含的元素集合的数量)。列表的最大长度为2^32- 1元素(4294967295，超过40亿每个元素的集合)。

Redis sorted set的内部使用HashMap和跳跃表(SkipList)来保证数据的存储和有序，HashMap里放的是成员到score的映射，而跳跃表里存放的是所有的成员，排序依据是HashMap里存的score,使用跳跃表的结构可以获得比较高的查找效率，并且在实现上比较简单。

```
    sortedset

    有序set是依靠分值来进行排序的。
    Zadd zset 10 a  :向zset集合中添加a元素，分值为10
    Zscore zset a  :获取集合zset中a元素的分值
    Zrange zset 0 -1 :获取zset中的所有元素
    Zrevrange zset 0 -1 ：按分值由高到低排列
    Zrevrang zset 0 1 :分值由高到低取2个
    Zrangbyscore zset 10 20 ：取zset中分值10 到20之间的，包括10和20
    Zrangbyscore zset (10 (20 :不包括10 和20 ，只是中间的值
    Zincrby zset 5 a ：对a的分值加5
    Zcard zset ：返回集合元素个数
    Zcount zset 10 20 ：返回zset中10-20之间的元素个数
    Zrem zset a :将a元素删除 
    Zremrangebyrank zset 0 1:将下标为0-1的元素删掉
    Zremrangebyscore zset 10 20 ：分数为10-20之间的删掉
    Zadd zset +info x ：x的分值为正无穷
    Zadd zset -info x ：x的分值为负无穷
```

### 5.redis的持久化

#### 5.1原因

因为在应用中redis来作为缓存，将用户的请求首先在redis查找，如果能找到直接返回，如果没有找到，就到mysql中查找，但是如果redis挂了之后，那么所以请求都上mysql去了，在并发量稍大一点的系统中，mysql瞬间就会挂了。

如果在redis挂了之后，能够快速的将数据恢复出来的话，那么mysql前面就有redis来进行支撑，不至于mysql宕掉。所以在这时redis挂了之后能快速的恢复就至关重要。或者恢复了之后要有缓存数据。

#### 5.2持久化策略

如果我们想要redis仅仅作为纯内存的缓存来用，那么可以禁止RDB和AOF所有的持久化机制。

##### 5.2.1RDB

将redis中数据，每隔一定的周期将这时内存中的快照数据保存到磁盘上，我们可以将这个rdb文件拷贝到其他主机进行恢复，或者我们可以将rdb这种数据存储在远端的云存储设备上，当redis宕机之后拉取一份rdb文件进行恢复即可。

RDB持久化机制，对redis中的数据执行周期性的持久化。

**优点：**

（1）RDB会生成多个数据文件，每个数据文件都代表了某一个时刻中redis的数据，这种多个数据文件的方式，非常适合做冷备，可以将这种完整的数据文件发送到一些远程的安全存储上去，比如说Amazon的S3云服务上去，在国内可以是阿里云的ODPS分布式存储上，以预定好的备份策略来定期备份redis中的数据。

RDB也可以做冷备，生成多个文件，每个文件都代表了某一个时刻的完整的数据快照。

AOF也可以做冷备，只有一个文件，但是你可以，每隔一定时间，去copy一份这个文件出来。

RDB做冷备，优势在哪儿呢？由redis去控制固定时长生成快照文件的事情，比较方便; 

AOF，还需要自己写一些脚本去做这个事情，各种定时。

RDB数据做冷备，在最坏的情况下，提供数据恢复的时候，速度比AOF快

（2）RDB对redis对外提供的读写服务，影响非常小，可以让redis保持高性能，因为redis主进程只需要fork一个子进程，让子进程执行磁盘IO操作来进行RDB持久化即可。

RDB，每次写，都是直接写redis内存，只是在一定的时候，才会将数据写入磁盘中
AOF，每次都是要写文件的，虽然可以快速写入os cache中，但是还是有一定的时间开销的,速度肯定比RDB略慢一些

（3）相对于AOF持久化机制来说，直接基于RDB数据文件来重启和恢复redis进程，更加快速

AOF，存放的指令日志，做数据恢复的时候，其实是要回放和执行所有的指令日志，来恢复出来内存中的所有数据的。

RDB，就是一份数据文件，恢复的时候，直接加载到内存中即可。

结合上述优点，**RDB特别适合做冷备份，冷备**。

**缺点：**

（1）如果想要在redis故障时，尽可能少的丢失数据，那么RDB没有AOF好。

一般来说，RDB数据快照文件，都是每隔5分钟，或者更长时间生成一次，这个时候就得接受一旦redis进程宕机，那么会丢失最近5分钟的数据。

这个问题，也是rdb最大的缺点，就是不适合做第一优先的恢复方案，如果你依赖RDB做第一优先恢复方案，会导致数据丢失的比较多。

（2）RDB每次在fork子进程来执行RDB快照数据文件生成的时候，如果数据文件特别大，可能会导致对客户端提供的服务暂停数毫秒，或者甚至数秒。

一般不要让RDB的间隔太长，否则每次生成的RDB文件太大了，对redis本身的性能可能会有影响的。

**配置：**

rdb:

save 900 1      // 900内,有1条写入,则产生快照 

save 300 1000   // 如果300秒内有1000次写入,则产生快照

save 60 10000  // 如果60秒内有10000次写入,则产生快照(这3个选项都屏蔽,则rdb禁用)

##### 5.2.2AOF

将用户操作的每条命令写到一个文件中，然后将这个文件中命令在重新执行一遍。

AOF机制对每条写入命令作为日志，以append-only的模式写入一个日志文件中，在redis重启的时候，可以通过回放AOF日志中的写入指令来重新构建整个数据集。

**优点：**

（1）AOF可以更好的保护数据不丢失，一般AOF会每隔1秒，通过一个后台线程执行一次fsync操作，最多丢失1秒钟的数据；

每隔1秒，就执行一次fsync操作，保证os cache中的数据写入磁盘中；

redis进程挂了，最多丢掉1秒钟的数据。

（2）AOF日志文件以append-only模式写入，所以没有任何磁盘寻址的开销，写入性能非常高，而且文件不容易破损，即使文件尾部破损，也很容易修复。

（3）AOF日志文件即使过大的时候，出现后台重写操作，也不会影响客户端的读写。因为在rewrite log的时候，会对其中的日志进行压缩，创建出一份需要恢复数据的最小日志出来。再创建新日志文件的时候，老的日志文件还是照常写入。当新的merge后的日志文件ready的时候，再交换新老日志文件即可。

（4）AOF日志文件的命令通过非常可读的方式进行记录，这个特性非常适合做灾难性的误删除的紧急恢复。
比如某人不小心用flushall命令清空了所有数据，只要这个时候后台rewrite还没有发生，那么就可以立即拷贝AOF文件，将最后一条flushall命令给删了，然后再将该AOF文件放回去，就可以通过恢复机制，自动恢复所有数据。

**缺点：**

（1）对于同一份数据来说，AOF日志文件通常比RDB数据快照文件更大。

（2）AOF开启后，支持的写QPS会比RDB支持的写QPS低，因为AOF一般会配置成每秒fsync一次日志文件，
当然，每秒一次fsync，性能也还是很高的；

如果你要保证一条数据都不丢，也是可以的，AOF的fsync设置成没写入一条数据，fsync一次，那就完蛋了，redis的QPS大降。

（3）以前AOF发生过bug，就是通过AOF记录的日志，进行数据恢复的时候，没有恢复一模一样的数据出来。

所以说，类似AOF这种较为复杂的基于命令日志/merge/回放的方式，比基于RDB每次持久化一份完整的数据快照文件的方式，更加脆弱一些，容易有bug。不过AOF就是为了避免rewrite过程导致的bug，因此每次rewrite并不是基于旧的指令日志进行merge的，而是基于当时内存中的数据进行指令的重新构建，这样健壮性会好很多。

（4）唯一的比较大的缺点，其实就是做数据恢复的时候，会比较慢，还有做冷备，定期的备份，不太方便，可能要自己手写复杂的脚本去做，做冷备不太合适。

**配置：**

aof:默认是不打开的。

appendonly no # 是否打开 aof日志功能

appendfsync always   # 每1个命令,都立即同步到aof. 安全,速度慢

appendfsync everysec # 折衷方案,每秒写1次

appendfsync no      # 写入工作交给操作系统,由操作系统判断缓冲区大小,统一写入到aof. 一般是30秒，同步频率低,速度快

no-appendfsync-on-rewrite  yes: # 正在导出rdb快照的过程中,要不要停止同步aof

auto-aof-rewrite-percentage 100 #aof文件大小比起上次重写时的大小,增长率100%时,重写

auto-aof-rewrite-min-size 64mb #aof文件,至少超过64M时,重写

手动执行:bgrewriteaof



通过RDB或AOF，都可以将redis内存中的数据给持久化到磁盘上面来，然后可以将这些数据备份到别的地方去，比如说阿里云，云服务。

如果redis挂了，服务器上的内存和磁盘上的数据都丢了，可以从云服务上拷贝回来之前的数据，放到指定的目录中，然后重新启动redis，redis就会自动根据持久化数据文件中的数据，去恢复内存中的数据，继续对外提供服务。

如果同时使用RDB和AOF两种持久化机制，那么在redis重启的时候，会使用AOF来重新构建数据，因为AOF中的数据更加完整。

##### 5.2.3RDB和AOF到底该如何选择

（1）不要仅仅使用RDB，因为那样会导致你丢失很多数据。

（2）也不要仅仅使用AOF，因为那样有两个问题，第一，你通过AOF做冷备，没有RDB做冷备，来的恢复速度更快; 第二，RDB每次简单粗暴生成数据快照，更加健壮，可以避免AOF这种复杂的备份和恢复机制的bug。

（3）综合使用AOF和RDB两种持久化机制，用AOF来保证数据不丢失，作为数据恢复的第一选择; 用RDB来做不同程度的冷备，在AOF文件都丢失或损坏不可用的时候，还可以使用RDB来进行快速的数据恢复。

### 6.aof重写

AOF 持久化是通过保存被执行的写命令来记录数据库状态的，所以AOF文件的大小随着时间的流逝一定会越来越大；影响包括但不限于：对于Redis服务器，计算机的存储压力；AOF还原出数据库状态的时间增加；

为了解决AOF文件体积膨胀的问题，Redis提供了AOF重写功能：Redis服务器可以创建一个新的AOF文件来替代现有的AOF文件，新旧两个文件所保存的数据库状态是相同的，但是新的AOF文件不会包含任何浪费空间的冗余命令，通常体积会较旧AOF文件小很多。

AOF重写并不需要对原有AOF文件进行任何的读取，写入，分析等操作，这个功能是通过读取服务器当前的数据库状态来实现的。

aof_rewrite函数可以创建新的AOF文件，但是这个函数会进行大量的写入操作，所以调用这个函数的线程将被长时间的阻塞，因为Redis服务器使用单线程来处理命令请求；所以如果直接是服务器进程调用AOF_REWRITE函数的话，那么重写AOF期间，服务器将无法处理客户端发送来的命令请求；

Redis不希望AOF重写会造成服务器无法处理请求，所以Redis决定将AOF重写程序放到子进程（后台）里执行。这样处理的最大好处是： 

- 子进程进行AOF重写期间，主进程可以继续处理命令请求；
- **子进程带有主进程的数据副本**，使用子进程而不是线程，可以避免在锁的情况下，**保证数据的安全性**。

**使用子进程进行AOF重写的问题：**

子进程在进行AOF重写期间，服务器进程还要继续处理命令请求，而新的命令可能对现有的数据进行修改，这会让当前数据库的数据和重写后的AOF文件中的数据不一致。

**如何修正：**

为了解决这种数据不一致的问题，Redis增加了一个AOF重写缓存，这个缓存在fork出子进程之后开始启用，Redis服务器主进程在执行完写命令之后，会同时将这个写命令追加到AOF缓冲区和AOF重写缓冲区。即子进程在执行AOF重写时，主进程需要执行以下三个工作： 

- 执行client发来的命令请求；

- 将写命令追加到现有的AOF文件中；

- 将写命令追加到AOF重写缓存中。

  ![æå¡å¨åæ¶å°å½ä"¤åéå°AOFæä"¶åAOFéåç¼å²åº](https://img-blog.csdn.net/20170406165541919?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaGV6aGlxaWFuZzEzMTQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



AOF缓冲区的内容会定期被写入和同步到AOF文件中，对现有的AOF文件的处理工作会正常进行，从创建子进程开始，服务器执行的所有写操作都会被记录到AOF重写缓冲区中；

**完成AOF重写之后：**

当子进程完成对AOF文件重写之后，它会向父进程发送一个完成信号，父进程接到该完成信号之后，会调用一个信号处理函数，该函数完成以下工作：

- 将AOF重写缓存中的内容全部写入到新的AOF文件中；这个时候新的AOF文件所保存的数据库状态和服务器当前的数据库状态一致；

- 对新的AOF文件进行改名，原子的覆盖原有的AOF文件；完成新旧两个AOF文件的替换。

这两个步骤（信号处理函数执行期间）会造成主进程阻塞，在其他时候，AOF后台重写都不会对主进程造成阻塞，这将AOF重写对性能造成的影响降到最低。

### 7.redis安装

yum install gcc-c++

  tar -zxvf redis-3.2.12.tar.gz -C /opt/modules/

  cd redis-3.2.12/

  make

  cd src

   make install PREFIX=/opt/modules/redis

  cd /opt/modules/redis-3.2.12/utils

  cp redis_init_script /etc/init.d/下

  mv /etc/init.d/redis_init_script /etc/init.d/redis_6379
  修改redis_6379脚本的第6行的REDISPORT，设置为相同的端口号（默认就是6379）
  修改EXEC=/opt/modules/redis/bin/redis-server
​      CLIEXEC=/opt/modules/redis/bin/redis-cli

  创建两个目录：mkdir /etc/redis（存放redis的配置文件）

  mkdir -p /var/redis/6379（存放redis的持久化文件）

  cp /opt/modules/redis-3.2.12/redis.conf /etc/redis/

  mv /etc/redis/redis.conf /etc/redis/6379.conf

修改redis.conf中的部分配置为生产环境
​	daemonize	yes							让redis以daemon进程运行
​	pidfile		/var/run/redis_6379.pid 	设置redis的pid文件位置
​	port		6379						设置redis的监听端口号
​	dir 		/var/redis/6379				设置持久化文件的存储位置

启动redis，执行cd /etc/init.d, chmod 777 redis_6379，./redis_6379 start

redis-benchmark：做redis的性能测试
​	redis-check-aof：用来redis AOF文件是否损坏以及修复
​	redis-check-rdb：用来检测redis rdb文件是否损坏
​	redis-cli：redis的客户端
​	redis-sentinel：哨兵脚本 哨兵是用来做redis的高可用
​	redis-server：server启动脚本

### 8.主从架构与集群

#### 8.1主从

如果系统的QPS超过10W+，甚至是百万以上的访问，则光是Redis是不够的，但是Redis是整个大型缓存架构中，支撑高并发的架构非常重要的环节。

般高并发的应用，写的请求是比较少的，大量的请求都是读。所以我们要采用  **主从架构 + 读写分离**，来支撑10W+ 的读QPS的系统。

**规划：**

192.168.25.161   master

192.168.25.162  slave

1、要按照之前的安装redis的方式分别在161和162上装好redis

2、在161进行配置：
   	在/etc/redis/6379.conf中
   	requirepass hello  ：设置密码

  在162上进行配置：
  	 在/etc/redis/6379.conf中
   	 slaveof 192.168.25.161 6379
 	 masterauth hello

3、查看集群状态
  	info replication

总结：
  	1、单节点：
 	 2、主从+哨兵：存储量一般几个G
 	 3、redis cluster：海量数据存储+读写分离（在redis cluster不存在读写分离）

  部署方案：
 	 1、双机房部署
   	 A、搞2套redis集群
​    	 B、分配每个节点到不同的机房。

#### 8.2集群

![redis集群](\asserts\redis集群.png)

### 9.哨兵

#### 1 哨兵的作用

哨兵是redis集群架构中非常重要的一个组件，主要功能如下： 
1. 集群监控：负责监控redis master和slave进程是否正常工作 

2. 消息通知：如果某个redis实例有故障，那么哨兵负责发送消息作为报警通知给管理员 

3. 故障转移：如果master node挂掉了，会自动转移到slave node上 

4. 配置中心：如果故障转移发生了，通知client客户端新的master地址

#### 2 哨兵的核心知识

1. 故障转移时，判断一个master node是宕机了，需要大部分的哨兵都同意才行，涉及到了分布式选举的问题
2. 哨兵至少需要3个实例，来保证自己的健壮性
3. 哨兵 + redis主从的部署架构，是不会保证数据零丢失的，只能保证redis集群的高可用性

#### 3 sdown和odown

sdown和odown两种失败状态
sdown是主观宕机，就一个哨兵如果自己觉得一个master宕机了，那么就是主观宕机
odown是客观宕机，如果quorum数量的哨兵都觉得一个master宕机了，那么就是客观宕机
sdown达成的条件：如果一个哨兵ping一个master，超过了is-master-down-after-milliseconds指定的毫秒数之后，就主观认为master宕机
odown达成的条件：如果一个哨兵在指定时间内，收到了quorum指定数量的其他哨兵也认为那个master是sdown了，那么就认为是odown了，客观认为master宕机

#### 4 quorum和majority

quorum：确认odown的最少的哨兵数量
​	majority：授权进行主从切换的最少的哨兵数量
每次一个哨兵要做主备切换，首先需要quorum数量的哨兵认为odown，然后选举出一个哨兵来做切换，这个哨兵还得得到majority哨兵的授权，才能正式执行切换
如果quorum < majority，比如5个哨兵，majority就是3，quorum设置为2，那么就3个哨兵授权就可以执行切换，但是如果quorum >= majority，那么必须quorum数量的哨兵都授权，比如5个哨兵，quorum是5，那么必须5个哨兵都同意授权，才能执行切换

#### 5 为什么哨兵至少3个节点

哨兵集群必须部署2个以上节点。如果哨兵集群仅仅部署了个2个哨兵实例，那么它的majority就是2（2的majority=2，3的majority=2，5的majority=3，4的majority=2），如果其中一个哨兵宕机了，就无法满足majority>=2这个条件，那么在master发生故障的时候也就无法进行主从切换。

#### 6 脑裂以及redis数据丢失

主备切换的过程，可能会导致数据丢失 

```
两个配置配置减少数据丢失：
min-slaves-to-write 1
min-slaves-max-lag 10
```

（1）**异步复制导致的数据丢失** 

因为master -> slave的复制是异步的，所以可能有部分数据还没复制到slave，master就宕机了，此时这些部分数据就丢失了 。

解决方案：

有了min-slaves-max-lag这个配置，就可以确保说，一旦slave复制数据和ack延时太长，就认为可能master宕机后损失的数据太多了，那么就拒绝写请求，这样可以把master宕机时由于部分数据未同步到slave导致的数据丢失降低的可控范围内 。

（2）**脑裂导致的数据丢失** 

脑裂，也就是说，某个master所在机器突然脱离了正常的网络，跟其他slave机器不能连接，但是实际上master还运行着 ，此时哨兵可能就会认为master宕机了，然后开启选举，将其他slave切换成了master，这个时候，集群里就会有两个master，也就是所谓的脑裂。 

此时虽然某个slave被切换成了master，但是可能client还没来得及切换到新的master，还继续写向旧master的数据可能也丢失了，因此旧master再次恢复的时候，会被作为一个slave挂到新的master上去，自己的数据会清空，重新从新的master复制数据。

解决方案：

如果一个master出现了脑裂，跟其他slave丢了连接，那么上面两个配置可以确保说，如果不能继续给指定数量的slave发送数据，而且slave超过10秒没有给自己ack消息，那么就直接拒绝客户端的写请求，这样脑裂后的旧master就不会接受client的新数据，也就避免了数据丢失 ，上面的配置就确保了，如果跟任何一个slave丢了连接，在10秒后发现没有slave给自己ack，那么就拒绝新的写请求 ，因此在脑裂场景下，最多就丢失10秒的数据。

### 10.redis如何防止宕机

1、事前：做好redis的备份策略
​	         双机房
​	2、事中：用之前的备份的快速恢复
​	         1、备份在
​		 2、没有备份
​			      预热缓存
​	3、事后：服务降级 histrix

### 11.缓存满了算法

 LRU：最近最少使用
​	 redis提供的淘汰策略

### 12.Java

**1.redis.properties**

```properties
redis.host=192.168.25.162
redis.port=6379
redis.maxIdle=150
redis.maxActive=300
redis.blockWhenExhausted=true
redis.maxWait=10000
redis.testOnBorrow=true
redis.timeout=100000
```

**2.applicationContext-redis.xml**

```xml
   <!-- redis数据源 -->
    <bean id="poolConfig" class="redis.clients.jedis.JedisPoolConfig">
        <!-- 最大空闲数 -->
        <property name="maxIdle" value="${redis.maxIdle}" />
        <!-- 最大空连接数 -->
        <property name="maxTotal" value="${redis.maxActive}" />
        <!-- 最大等待时间 -->
        <property name="maxWaitMillis" value="${redis.maxWait}" />
        <!-- 连接超时时是否阻塞，false时报异常,ture阻塞直到超时, 默认true -->
         <property name="blockWhenExhausted" value="${redis.blockWhenExhausted}" /> 
        <!-- 返回连接时，检测连接是否成功 -->
        <property name="testOnBorrow" value="${redis.testOnBorrow}" />
    </bean>

    <!-- Spring-redis连接池管理工厂 -->
    <bean id="jedisConnectionFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
        <!-- IP地址 -->
        <property name="hostName" value="${redis.host}" />
        <!-- 端口号 -->
        <property name="port" value="${redis.port}" />
        <!-- 超时时间 默认2000-->
        <property name="timeout" value="${redis.timeout}" />
        <!-- 连接池配置引用 -->
        <property name="poolConfig" ref="poolConfig" />
        <!-- usePool：是否使用连接池 -->
        <property name="usePool" value="true"/>
    </bean>

    <!-- redis template definition -->
    <bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
        <property name="connectionFactory" ref="jedisConnectionFactory" />
        <property name="keySerializer">
            <bean class="org.springframework.data.redis.serializer.StringRedisSerializer" />
        </property>
        <property name="valueSerializer">
            <bean class="org.springframework.data.redis.serializer.JdkSerializationRedisSerializer" />
        </property>
        <property name="hashKeySerializer">
            <bean class="org.springframework.data.redis.serializer.StringRedisSerializer" />
        </property>
        <property name="hashValueSerializer">
            <bean class="org.springframework.data.redis.serializer.JdkSerializationRedisSerializer" />
        </property>
         <!--开启事务  -->  
        <property name="enableTransactionSupport" value="true"></property>  
    </bean>
    
    <bean id="redisManager" class="cn.kgc.util.RedisManager">
    	<property name="redisTemplate" ref="redisTemplate"></property>
    </bean>

</beans>

```

**3.RedisSessionDao**

```java
package cn.kgc.dao;

import java.io.Serializable;
import java.util.Collection;
import java.util.HashSet;
import java.util.Set;

import org.apache.shiro.session.Session;
import org.apache.shiro.session.UnknownSessionException;
import org.apache.shiro.session.mgt.eis.AbstractSessionDAO;

import cn.kgc.util.RedisManager;
import cn.kgc.util.SerializerUtil;

public class RedisSessionDao extends AbstractSessionDAO{

	private RedisManager redisManager;

    /** 
     * The Redis key prefix for the sessions  
     */  
    private static final String KEY_PREFIX = "shiro_redis_session:"; 

    @Override
    public void update(Session session) throws UnknownSessionException {
        this.saveSession(session);
    }

    @Override
    public void delete(Session session) {
        if (session == null || session.getId() == null) {
            return;
        }
        redisManager.del(KEY_PREFIX + session.getId());
    }

    @Override
    public Collection<Session> getActiveSessions() {
        Set<Session> sessions = new HashSet<Session>();  
        Set<byte[]> keys = redisManager.keys(KEY_PREFIX + "*");  
        if(keys != null && keys.size()>0){  
            for(byte[] key : keys){  
                Session s = (Session)SerializerUtil.deserialize(redisManager.get(SerializerUtil.deserialize(key)));  
                sessions.add(s);  
            } 
        }
        return sessions;  
    }

    @Override
    protected Serializable doCreate(Session session) {
        Serializable sessionId = this.generateSessionId(session);
        this.assignSessionId(session, sessionId);
        this.saveSession(session);
        return sessionId;
    }

    @Override
    protected Session doReadSession(Serializable sessionId) {
         if(sessionId == null){  
                return null;  
            }  

            Session s = (Session)redisManager.get(KEY_PREFIX + sessionId);  
            return s;  
    }

    private void saveSession(Session session) throws UnknownSessionException{
        if (session == null || session.getId() == null) {
            return;
        }
        //设置过期时间
        long expireTime = 1800000l;
        session.setTimeout(expireTime);
        redisManager.setEx(KEY_PREFIX + session.getId(), session, expireTime);
    }

    public void setRedisManager(RedisManager redisManager) {
        this.redisManager = redisManager;
    }

    public RedisManager getRedisManager() {
        return redisManager;
    }

}

```

**4.SerializerUtil**

```java
package cn.kgc.util;

import org.springframework.data.redis.serializer.JdkSerializationRedisSerializer;

public class SerializerUtil {

	private static final JdkSerializationRedisSerializer jdkSerializationRedisSerializer = new JdkSerializationRedisSerializer();

    /**
     * 序列化对象
     * @param obj
     * @return
     */
    public static <T> byte[] serialize(T obj){
        try {
            return jdkSerializationRedisSerializer.serialize(obj);
        } catch (Exception e) {
            throw new RuntimeException("序列化失败!", e);
        }
    }

    /**
     * 反序列化对象
     * @param bytes 字节数组
     * @param cls cls
     * @return
     */
    @SuppressWarnings("unchecked")
    public static <T> T deserialize(byte[] bytes){
        try {
            return (T) jdkSerializationRedisSerializer.deserialize(bytes);
        } catch (Exception e) {
            throw new RuntimeException("反序列化失败!", e);
        }
    }
}

```

**5.RedisManager**

```java
package cn.kgc.util;

import java.io.Serializable;
import java.util.Set;

import org.apache.shiro.dao.DataAccessException;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.connection.RedisConnection;
import org.springframework.data.redis.core.RedisCallback;
import org.springframework.data.redis.core.RedisTemplate;

import com.alibaba.druid.util.StringUtils;

public class RedisManager {

    private RedisTemplate<Serializable, Serializable> redisTemplate;
    
    

    public void setRedisTemplate(RedisTemplate<Serializable, Serializable> redisTemplate) {
		this.redisTemplate = redisTemplate;
	}

	/**
     * 添加缓存数据（给定key已存在，进行覆盖）
     * @param key
     * @param obj
     * @throws DataAccessException
     */
    public <T> void set(String key, T obj) throws DataAccessException{
        final byte[] bkey = key.getBytes();
        final byte[] bvalue = SerializerUtil.serialize(obj);
        redisTemplate.execute(new RedisCallback<Void>() {
            @Override
            public Void doInRedis(RedisConnection connection) throws DataAccessException {
                connection.set(bkey, bvalue);
                return null;
            }
        });
    }

    /**
     * 添加缓存数据（给定key已存在，不进行覆盖，直接返回false）
     * @param key
     * @param obj
     * @return 操作成功返回true，否则返回false
     * @throws DataAccessException
     */
    public <T> boolean setNX(String key, T obj) throws DataAccessException{
        final byte[] bkey = key.getBytes();
        final byte[] bvalue = SerializerUtil.serialize(obj);
        boolean result = redisTemplate.execute(new RedisCallback<Boolean>() {
            @Override
            public Boolean doInRedis(RedisConnection connection) throws DataAccessException {
                return connection.setNX(bkey, bvalue);
            }
        });

        return result;
    }

    /**
     * 添加缓存数据，设定缓存失效时间
     * @param key
     * @param obj
     * @param expireSeconds 过期时间，单位 秒
     * @throws DataAccessException
     */
    public <T> void setEx(String key, T obj, final long expireSeconds) throws DataAccessException{
        final byte[] bkey = key.getBytes();
        final byte[] bvalue = SerializerUtil.serialize(obj);
        redisTemplate.execute(new RedisCallback<Boolean>() {
            @Override
            public Boolean doInRedis(RedisConnection connection) throws DataAccessException {
                connection.setEx(bkey, expireSeconds, bvalue);
                return true;
            }
        });
    }

    /**
     * 获取key对应value
     * @param key
     * @return
     * @throws DataAccessException
     */
    public <T> T get(final String key) throws DataAccessException{
        byte[] result = redisTemplate.execute(new RedisCallback<byte[]>() {
            @Override
            public byte[] doInRedis(RedisConnection connection) throws DataAccessException {
                return connection.get(key.getBytes());
            }
        });
        if (result == null) {
            return null;
        }
        return SerializerUtil.deserialize(result);
    }

    /**
     * 删除指定key数据
     * @param key
     * @return 返回操作影响记录数
     */
    public Long del(final String key){
        if (StringUtils.isEmpty(key)) {
            return 0l;
        }
        Long delNum = redisTemplate.execute(new RedisCallback<Long>() {
            @Override
            public Long doInRedis(RedisConnection connection) throws DataAccessException {
                byte[] keys = key.getBytes();
                return connection.del(keys);
            }
        });
        return delNum;
    }

    public Set<byte[]> keys(final String key){
        if (StringUtils.isEmpty(key)) {
            return null;
        }
        Set<byte[]> bytesSet = redisTemplate.execute(new RedisCallback<Set<byte[]>>() {
            @Override
            public Set<byte[]> doInRedis(RedisConnection connection) throws DataAccessException {
                byte[] keys = key.getBytes();
                return connection.keys(keys);
            }
        });

        return bytesSet;
    }

}

```

### 13.一致性hash
![一致性hash](\asserts\一致性hash.png)
理解即可


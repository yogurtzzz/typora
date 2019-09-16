# Redis基础

redis是单线程的

## redis 命令

**====== string类型相关**

* set [key] [val]
* get [key]
* del [key]
* mset  [key1] [val1] [key2] [val2] ..
* mget  [key1] [key2] ..
* setnx [key] [val]
* incr [key]
* incrby [key] [increment]
* decr
* decrby

**========= hash类型相关**

* hset [key] [field] [val]
* hmset [key] [field1] [val1] [field2] [val2] ..
* hget [key] [field]  获取一个字段的值
* hmget [key] [field1] [field2] ..    获取多个字段的值
* hgetall [key]   获取所有字段的值  （结果是key value）
* hsetnx [key] [field] [val]
* hdel [key] [field1] [field2] ..  删除一个或多个字段
* hincrby [key] [field] [increment]  
* hexists [key] [field]  判断某个字段是否存在
* hkeys [key]    只取字段名
* hvals [key]   只取字段值
* hlen [key]   获取字段数量

* hdel [key] [field1] [field2] ..

**======== list类型相关**

* lpush [key] [val1] [val2] ..

* rpush [key] [val1] [val2] ..

* lrange [key] [start] [end]  : 取出start 到 end之间的所有元素 （包含start 和 end）

  `lrange class 0 -1` 表示获取class这个list的所有元素

* lpop [key] ：获取并弹出左边第一个元素

* rpop [key] 

* blpop [key] [timeout]：若没有值，则一直阻塞等待，当等待超过timeout的时间后，才返回null

* brpop [key] [timeout]

* llen [key] 获取列表中元素个数

* lrem [key] [count] [val]  ： 删除列表中**前count个**值为val的元素，count > 0时，从左边开始删，count<0时，从右边开始删，count=0时，删除所有值为val的元素

* lindex [key] [index] ：获得指定索引的元素值，index>0，表示从左边开始数，index<0，表示从右边开始数

* lset [key] [index] [val] ：设置指定索引的元素值

* ltrim [key] [start] [end] ：只保留列表指定片段

* linsert [key] BEFORE|AFTER [pivot] [val] ：向列表中插入元素，插入在元素**之前**或者**之后**

* rpoplpush [source] [destination] ：对source列表执行rpop，并将得到的元素插入到destination列表中



**======== set类型相关**

* sadd [key] [member1] [member2] .. ：添加元素
* srem [key] [member1] [member2] .. ：删除元素
* smembers [key] ：获得set中全部元素
* sismember [key] [member]：判断member是否在key的集合中
* sdiff [key1] [key2] .. ：差集运算，计算属于key1，但不属于key2的集合
* sinter [key1] [key2] .. 交集运算，计算属于key1，同时也属于key2的集合
* sunion [key1] [key2] .. ：并集运算
* scard [key] ：获取集合的元素总数
* spop [key]  (count)：从集合中随机pop一个元素出来，可选参数count，指定弹出个数

**======= zset类型相关**

* zadd [key] [score] [member]  ( [score] [member] ).. ：添加元素
* zrange [key] [start] [end]  ( [WITHSCORES] )：获得排名在某个范围内的元素列表（从小到大）

* zrevrange : （从大到小）  例如获得成绩排名的前3：`zrevrange class 0 2 WITHSCORES`
* zscore [key] [member] ：获取某元素的分数，例如获得某人的分数：`zscore class hby`
* zrem [key] [member] [member].. ：删除元素
* zrangebyscore [key] [min] [max] ：获得指定分数范围的元素
* zincrby [key] [increment] [member]：增加某个元素的分数
* zcard [key]：获得set中元素的数量
* zcount [key] [min] [max]：获得指定分数范围的元素数量，如统计成绩及格的人数 `zcount class 60 100`
* zremrangebyrank [key] [start] [end] ：删除某一排名范围内的元素
* zremrangebyscore [key] [min] [max]：删除某一分数范围内的元素
* zrank [key] [member] ：获得元素的排名（从小到大）
* zrevrank ：（从大到小）

**======通用命令**

* keys [pattern]  ：返回满足匹配模式的所有key
* del [key] ：删除某一个key的数据
* exists [key] ：判断某个key是否存在
* **expire** [key] [seconds] ：设置某个key的失效时间（单位秒），超过该时间后key会失效（自动删除）
* pexpire [key] [mills] ：设置某个key的失效时间（单位毫秒）
* ttl [key]：查看某个key的剩余生存时间（time to live）（为-1表示没有设置失效时间，-2表示元素已不存在）
* persist [key] ：移除失效时间的限制（必须在key还未被删除时）

* rename [oldKey] [newKey] ：重命名
* type [key] ：显示指定的key的数据类型



## redis  数据类型

* string：也可存储对象数据，对象属性的查询比较方便
* hash ： 适合存储对象数据（对象属性经常发生增删改的数据）
* list：内部是一个双向链表，是有序的（指**插入顺序**） ——>  适合**消息队列**的场景
* set：其中的数据是不重复的，且没有顺序
* zset：在set基础上，是有顺序的（指**自然顺序**），每个元素都会关联一个分数（sorted set），元素会按关联的分数进行排序 ——> 适合**销售量排行榜**的场景





## redis 消息模式

* **队列模式**：使用list类型的lpush和rpop实现消息队列

  消息接收方如果不知道队列中是否有消息，会一直发送rpop命令，这样每一次都会建立连接

  可以使用brpop命令，会使得如果从队列中取不出数据，则会一直阻塞，超出指定时间后，会返回null

* **发布订阅模式**：

  发布者发送消息，订阅者接收消息

  * 订阅者订阅一个频道：`subscribe yogurtChannel`

    ![](https://www.runoob.com/wp-content/uploads/2014/11/pubsub1.png)

  * 发布者发布消息到频道：`publish yogurtChannel "我是yogurt"`

    ![](https://www.runoob.com/wp-content/uploads/2014/11/pubsub2.png)

# Redis 事务

通过`multi` ，`exec`，`discard`，`watch` 来完成

* redis的单个命令都是**原子性**的，事务性的对象是**命令的集合**
* redis将命令集合序列化，并保证处于同一事务的命令集合，连续不被打断地执行
* redis不支持回滚

## 事务命令

* **multi**

  用于标记事务块的开始，redis会将后续的命令，逐个放入队列中，然后使用`exec`来原子化地执行这个命令队列

* **exec**

  执行所有先前放入队列的命令，后恢复正常的连接状态

* **discard**

  清除所有先前放入队列的命令，后恢复正常的连接状态

* **watch**

  监控某个key，若在开启监控到执行命令之间，有其他redis客户端并发地对被监控的key做了修改，则事务会失败（整个命令集合都会一起失败）

* **unwatch**

  清除对某个key的监控

## 事务失败处理

* 语法错误（编译期）

  命令队列的命令全部都不会执行
  
  > multi
  >
  > set v vage
  >
  > set v1
>
  > exec

* 运行错误

  运行期间某个命令失败，不会影响其他命令的执行

  > multi
  >
  > set v vae 
  >
  > lpush v 1 2 3
  >
  > exec

  运行结果

  > 1) OK
  >
  > 2) (error) WRONGTYPE .....

  

* redis不支持事务回滚
  * 大多数事务失败，是因为**语法错误**，或**类型错误**，这在开发阶段是可预见的
  * 正是因为



# Redis持久化

redis是一个内存数据库，为了保证数据的持久化，其提供了2种持久化方案

* `RDB`方式（默认）
* `AOF`方式

## RDB方式

是通过**快照**（Snapshot）完成的，在**符合一定条件**时，会自动将内存中的数据进行快照，并持久化到硬盘。

redis默认使用RDB方式进行持久化，实际操作过程是fork一个子进程，先将数据集写入临时文件，写入成功后，再替换之前的文件，用二进制压缩存储。RDB是Redis默认的持久化方式，会在对应的目录下生产一个dump.rdb文件，重启会通过加载dump.rdb文件恢复数据。

* 触发快照的时机：
  * 自定义配置的快照规则
  * 执行`save`或`bgsave`命令
  * 执行`flushall`命令
  * 执行主从复制操作

* 说明：
  * redis启动后，会读取`RDB`文件，将数据从硬盘载入内存
  * 根据**数据量大小与结构**，**服务器性能**的不同，需要设置不同的快照条件

* 设置快照规则

  * 设置触发条件

    `save [seconds] [changes]` 

    每seconds秒内，至少有changes个key被更改，则进行快照，如`save 300 10`

  * 配置RDB快照文件的存放位置

    `dir [path]`

    设为默认即可，`dir ./`

  * 配置RDB快照文件的名称

    `dbfilename [name]`

    设为默认即可，`dbfilename dump.rdb`

* RDB快照的基本原理
  * redis（server）调用系统中的`fork`函数，复制一份当前进程的副本（子进程）
  * 父进程继续接受并处理客户端发来的命令，子进程开始将内存中的数据写入到硬盘中的**临时文件**
  * 当子进程写入完成所有数据后，会用该临时文件，**替换掉旧的RDB文件**，至此，快照完成



* 优缺点
  * 优点
    * 可以最大化Redis的性能。父进程只需fork一个子进程，这个子进程会处理持久化操作，父进程不会进行任何IO操作
  * 缺点
    * 数据安全性较低。RDB是间隔一段时间进行一次持久化，若持久化之间redis发生故障，则这个期间的数据会丢失
    * 由于需要fork子进程来协助完成持久化，当数据集较大时，会导致整个服务器停止服务较长时间（几百毫秒甚至1秒）

## AOF方式

append only file  ，默认是没有开启的

开启AOF持久化后，每执行一条**更改数据的命令**，就会将该命令写入到硬盘中的AOF文件中，这样会明显降低Redis的性能，可使用较快的硬盘来提高AOF的性能



修改`redis.conf`文件

* `appendonly yes`    开启AOF
* `dir ./`
* `appendfilename appendonly.aof`



每次redis更改数据时，AOF机制都会将命令记录到AOF文件中，但由于操作系统的缓存机制，数据时先进入到**硬盘缓存**，再通过**硬盘缓存机制**去刷新，并保存到文件

```properties
# 每次执行写入都会进行同步（将缓冲区数据刷新到磁盘），每次执行write后都会调用fsync，会极大削弱redis性能
appendfsync always
# 每秒调用一次fsync
appendfsync everysec
# write后不会调用fsync ，由操作系统自动调度刷新磁盘，此时性能是最好的
appendfsync no
```





设置密码：

* 启动redis-server

* 启动redis-cli

  在客户端设置密码

  ```
  auth xx    
  先查看是否设置了密码
  (error) ERR Client sent AUTH, but no password is set
  出现这样的信息表示还没有设置密码
  config set requirepass "123456"
  设置完成后，当前登录的redis客户端没有权限访问了
  需要重启客户端
  ./redis-cli -a 123456
  或者先登录 ./redis-cli
  再输入密码
  auth 123456
  ```

  但是这样设置，只能生效一次，redis-server重启后，客户端连接仍然无需密码



# 高级应用

## Bitmap

可以用来压缩空间  [参考文献](https://www.jianshu.com/p/62cf39db5c2f)





在java中使用Jedis来操作redis

[菜鸟教程](https://www.runoob.com/redis/redis-java.html)
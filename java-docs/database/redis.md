## Redis

### 1. 简介
> Redis(Remote Dictionary Server)是完全开源的，遵守BSD协议，是一个高性能的<key-value>数据库。
>> 三个特点:
>> - Redis 支持数据持久化. 可以将内存中的数据保持在磁盘中, 重启后可再次使用
>> - Redis 支持 key-val, list, set, ZSet, hash 等数据结构存储
>> - Redis 支持数据备份, master-slave 模式数据备份
>>> 零碎基础知识
>>> - 单进程 (*) 单进程处理客户端的请求, 对读写事件响应是通过 epoll 函数而包装来操作. Redis 实际处理速度完全依靠主进程执行效率
>>> - 默认 16 个数据库, 类似数组下标从 0 开始, 初始默认使用 0 号库
>>> - select 切换数据库
>>> - dbSize 查看当前数据库的 key 的数量
>>> - flushDB 清空当前数据库
>>> - flushAll 删全部库
>>> - 统一密码管理, 16 个库同一个密码
>>> - Redis 索引是从 0 开始
>>> - 默认端口 6379

### 2. 数据类型
> - String (字符串)
> - Hash (哈希, 类似 Java HashMap)
> - List (列表)
> - Set (集合)
> - ZSet (Sorted Set: 有序集合)
>> #### 2.1 String
>>     1. 一个 key 对应一个 value
>>     2. 二进制安全. 可以包含任何数据, 包括图片或者序列化对象
>>     3. value 最多可以是 512M
>> #### 2.2 Hash
>>     1. string 类型的 field 和 value 的映射表
>> #### 2.3 List
>>     1. 底层是一个链表
>>     2. 按照插入顺序排序, 可以在头部或者尾部插入数据
>> #### 2.4 Set
>>     1. string 类型的无序集合, 是通过 HashTable 实现的
>> #### 2.5 ZSet (Sorted Set)
>>     1. string 类型的集合
>>     2. 与 Set 不同的是: 每个元素都会关联一个 double 类型的分数
>>     3. Redis 通过分数来为集合中的成员进行从小到大排序
>>     3. zset 成员唯一, 但是分数 (score) 可以重复

### 3. 持久化 (Persistence)
> Redis 是一个 key-value 的内存 NoSQL 数据库, 所有的数据都保存在数据库里
  <br>对于只把 Redis 当缓存来用的项目来说，数据消失或许问题不大，重新从数据源把数据加载进来就可以了，但如果直接把用户提交的业务数据存储在 Redis 当中，把Redis作为数据库来使用，在其放存储重要业务数据，那么Redis的内存数据丢失所造成的影响也许是毁灭性。
  <br>为了避免内存中数据丢失，Redis提供了对持久化的支持，我们可以选择不同的方式将数据从内存中保存到硬盘当中，使数据可以持久化保存。
>> #### 3.1 RDB (Redis Database)
>>> ##### 3.1.1 简介
>>>     a. 在指定的时间间隔内将内存中的数据集快照写入磁盘, 也就是 Snapshot 快照, 恢复时将快照文件直接读到内存中
>>>     b. Redis 会单独创建 (fork) 一个子进程来进行持久化, 会将数据写入到一个临时文件中, 待持久化过程结束, 再用这个临时文件替换上次持久化好的文件.
>>>     c. 整个过程, 主进程没有 IO 操作
>>>     d. 如果需要进行大规模数据恢复, 且对于数据恢复的完整性不敏感. RDB > AOF. 但是 RDB 缺点是最后一次持久化后的数据可能会丢失
>>> ##### 3.1.2 DB File
>>>     a. RDB 保存的是 dump.rdb 文件
>>>     b. save <seconds> <changes> xx 秒内 key 有xx次更改, 就保存 DB 到 Disk
>>>     c. 如果 SHUTDOWN 也会立即生成一个 dump.rdb
>>>     d. 冷拷贝方便使用
>>>     e. 停止保存 save ""
>>> ##### 3.1.3 Fork
>>>     复制一个与当前进程一样的进程, 新进程所有数值和原进程一致
>>> ##### 3.1.4 Snapshot 快照
>>>     a. SAVE: save 只管保存, 其他的不管, 全部阻塞
>>>     b. BGSAVE: Redis 会在后台异步进行快照操作, 快照同时还可以响应客户端请求, 可以通过 lastsave 命令获取最后一次成功执行快照的时间
>>>     c. 执行 FLUSHALL 也会生成 dump.rdb 但是是空的
>>> ##### 3.1.5 优势
>>>     a. 适合大规模的数据恢复
>>>     b. 对数据完整性和一致性要求不高
>>>     c. 与AOF方式相比，通过RDB文件恢复数据比较快
>>>     d. RDB文件非常紧凑，适合于数据备份
>>>     e. 通过 RDB 进行数据备，由于使用子进程生成，所以对 Redis 服务器性能影响较小
>>> ##### 3.1.6 劣势
>>>     a. 在一定时间间隔做一次备份, 如果 Redis 意外 Shutdown, 就会丢失最后一次快照后的所有修改
>>>     b. Fork 的时候, 内存中的数据克隆了一份, 2倍膨胀性
>>>     c. 使用 save 命令会造成服务器阻塞，直接数据同步完成才能接收后续请求。
>>>     d. 使用 bgsave 命令在 forks子进程时，如果数据量太大，forks 的过程也会发生阻塞，另外，forks 子进程会耗费内存
>>> ![Markdown](https://i.loli.net/2021/04/27/Qr389lcHGe4w6zM.png)
>> #### 3.2 AOF (Append Only File)
>>> ##### 3.2.1 简介
>>>     a. 以日志的形式来记录每个写操作, 将 Redis 执行过的所有写指令记录下来 (读操作不记录) 只许追加文件但不可以改写文件
>>>     b. Redis 启动之初会读取改文件重新创建数据
>>> ##### 3.2.2 保存文件
>>>     a. appendfilename = appendonly.aof
>>>     b. APPEND ONLY MODE - good enough
>>>     c. 如果既有 dump.rdb 又有 appendonly.aof, 可以和平共存, 先加载 appendonly.aof (with better durability guarantees)
>>>     d. Appendfsync : default 是 Everysec异步操作, 每秒记录, Always 同步持久化
>>> ##### 3.2.3 Rewrite 重写机制
>>>> ###### 3.2.3.1 简介
>>>>     AOF 采用文件追加方式, 文件会越拉越大为避免出现此种情况, 新增重写机制, 当 AOF 文件大小超过设定的阈值, Redis 就会情动 AOF 文件内容压缩, 只保留可以恢复数据的最小指令集, 可以使用命令 bgrewriteaof
>>>> ###### 3.2.3.2 重写原理
>>>>     AOF 文件大小增长而过大时, 会 fork 出一条新进程来将文件重写, 也是先写临时文件然后 rename, 遍历新进程的内存中的数据. AOF 重写方式也是异步操作
>>>> ###### 3.2.3.3 触发机制
>>>>     Redis 会记录上次重写时的 AOF 大小, 默认配置是当 AOF 文件大小是上次 rewrite 后大小的一倍并且文件大于 64MB 触发
>>> ##### 3.2.4 优势
>>>     a. AOF 只是追加日志文件，因此对服务器性能影响较小，速度比 RDB 要快，消耗的内存较少
>>>     b. 每秒同步: appendfsync always 同步持久化, 每次发生数据变更会被立即记录到磁盘 性能较差但是完整性好
>>>     c. 每修改同步: appendfsync everysec 异步操作, 每秒记录, 如果一秒内宕机, 有数据丢失
>>>     d. 不同步: appendfsync no
>>> ##### 3.2.5 劣势
>>>     a. 相同数据集的数据而言 aof 文件要大于 rdb 文件, 恢复速度慢于 rdb
>>>     b. aof 运行效率慢于 rdb, 同步策略效率较好, 不同步效率和 rdb 相同
>>> ![Markdown](https://i.loli.net/2021/04/27/gck3r7pLmNUZ64q.png)
>> #### 3.3 总结
>>     a. RDB 持久化方式能够在指定的时间间隔能付InIDE数据进行快照存储
>>     b. AOF 持久化方式记录每次对服务器写的操作, 当服务器重启的时候会重新执行这些命令来恢复原始的数据, AOF 命令以 Redis 协议追加保存每次写的操作到文件末尾
>>     c. Redis 还能对 AOF 文件进行后台重写, 使得文件体积不至于过大
>>     d. 只做缓存: 如果只希望你的数据在服务器运行的时候存在, 可以不使用持久化方式
>>     e. 同时开启两种持久化方式
>>        Redis 重启的时候会优先载入 AOF 文件来回复原始的数据, 因为通常情况下, AOF 保存的数据更完整
>>        RDB 数据不实, 同时使用两者时服务器重启也只会找 AOF 文件
>>     f. 当 RDB 与 AOF 两种方式都开启时，Redis会优先使用AOF日志来恢复数据，因为AOF保存的文件比RDB文件更完整
>> ![Markdown](https://i.loli.net/2021/04/27/3LQRoE1MGsd582e.png)



```java
 public Boolean lock(String key,String value,Long timeOut){
     String var1 = jedis.set(key,value,"NX","EX",timeOut); //加锁，设置超时时间 原子性操作
     if(LOCK_SUCCESS.equals(var1)){
         return true;
     }
     return false;
 }
```























```html
    key *   
    exists key # 判断某个 key 是否存在   
    move key db # 将 key 移动到另一个库        
    expire key # 给 key 设置过期时间       
    ttl key # 查看还有多长时间 key 过期, -1 表示永不过期, -2 表示已经过期    
    type key # 查看 key 是什么类型
    get k1 # 有对应的键值
    set k1 v1 # 有对应的键值, 覆盖 value
    del key # 删除 key
```

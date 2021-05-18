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

### 4. 分布式事务
> 可以一次执行多个命令, 本质是一组命令的集合, 一个事务中的所有命令都会序列化, 按顺序的串行化执行而不会被其他的命令插入 (不许加塞)
> <br>按照传统的系统架构，下单、扣库存等等，这一系列的操作都是一在一个应用一个数据库中完成的，也就是说保证了事务的ACID特性。如果在分布式应用中就会涉及到跨应用、跨库。这样就涉及到了分布式事务，就要考虑怎么保证这一系列的操作要么都成功要么都失败。保证数据的一致性。
> <br>一个队列中, 一次性, 顺序性, 排他性的执行一系列命令
> <br>1. 正常执行
> <br>2. 放弃事务
> <br>3. “全体连坐”
> <br>4. “冤头债主”
> <br>5. watch 监控
>> <br> 悲观锁: 并发性差, 一致性很好. 锁住整张表
>> <br> 乐观锁: 并发性可以. 每一行会有一个 version 值
>> <br> 先开启监控 WATCH 再开启 MULTI , 保证两笔金额变动在同一个事务内
>  WATCH 指令, 类似于乐观锁, 事务提交时, 如果 Key 已经被其他的客户端改变, 整个事务队列不会执行
>  <br>通过 WATCH 命令在事务执行之前监控了多个 Keys, 倘若在WATCH 之后有任何 Key 的值发生了变化, EXEC 命令执行的事物都将被放弃, 同时返回 Nullmulti-bulk 应答以通知调用者事务执行失败
>> #### 4.1 三个阶段
>>     a. 开启: 以 MULTI 开始一个事务
>>     b. 入队: 将多个命令入队到事务中, 接到这些命令并不会立即执行, 而是放到等待执行的事务队列里面
>>     c. 执行: 由 EXEC 命令触发事务
>> #### 4.2 特性
>>     a. 单独的隔离操作: 事务中的所有命令都会序列化, 按顺序的执行. 事务在执行的过程中, 不会被其他客户端发送来的命令请求所打断
>>     b. 没有隔离级别的概念: 队列中的命令没有提交之前都不会实际的执行, 因为事务提交之前任何指令都不会被实际执行, 也就不存在”事务内的查询要看到事务里的更新, 在事务外查询不能看到”的问题
>>     c. 不保证原子性: Redis 同一个事务中如果有一条命令执行失败, 其后的命令仍然会被执行, 没有回滚
>> #### 4.3 常见的分布式事务锁
>>     a. 数据库级别的锁(乐观锁，基于加入版本号实现 && 悲观锁，基于数据库的 for update 实现)
>>     b. Redis ，基于 SETNX、EXPIRE 实现
>>     c. Zookeeper，基于InterProcessMutex 实现
>>     d. Redisson，lcok、tryLock（背后原理也是Redis）
>> #### 4.4 Redis 分布式事务锁搭建模式
>>> 单机: 只要一台Redis服务器，挂了就无法工作了
>>> <br>主从: 是备份关系， 数据也会同步到从库，还可以读写分离
>>> <br>哨兵: master挂了，哨兵就行选举，选出新的master，作用是监控主从，主从切换
>>> <br>集群: 高可用，分散请求。目的是将数据分片存储，节省内存
>> #### 4.5 Redis 分布式锁原理
>>> ##### 4.5.1 原理
>>>     a. 互斥性: 保证同一时间只有一个客户端可以拿到锁
>>>     b. 安全性: 只有加锁的服务才能有解锁权限，也就是不能让客户端A加的锁，客户端B、C 都可以解锁
>>>     c. 避免死锁
>>>     d. 保证加锁与解锁操作是原子性操作
>>>        这个其实属于是实现分布式锁的问题，假设a用redis实现分布式锁, 
>>>        假设加锁操作，操作步骤分为两步：1，设置key set（key，value） 2，给key设置过期时间
>>>        假设现在a刚实现set后，程序崩了就导致了没给key设置过期时间就导致key一直存在就发生了死锁
>>> ##### 4.5.2 加锁
>>>     SET key value NX EX timeOut
>>>     NX：只有这个key不存才的时候才会进行操作，即 if not exists；
>>>     EX：设置key的过期时间为秒，具体时间由第5个参数决定
>>>     timeOut：设置过期时间保证不会出现死锁【避免宕机死锁】
>>> 代码实现
```java
 public Boolean lock(String key,String value,Long timeOut){
    //加锁，设置超时时间 原子性操作
    String var1 = jedis.set(key,value,"NX","EX",timeOut); 
    if(LOCK_SUCCESS.equals(var1)){
        return true;
    }
    return false;
 }
```
>>>     当前没有锁（key不存在），那么就进行加锁操作，并对锁设置个有效期，同时value表示加锁的客户端。
>>>     已有锁存在，不做任何操作。
>>> ##### 4.5.3 解锁
>>> 代码实现
```java
 public Boolean redisUnLock(String key, String value) {
     String luaScript = "if redis.call('get',KEYS[1]) == ARGV[1] then return redis.call('del',KEYS[1]) else  return 0 end";
     Object var2 = jedis.eval(luaScript, Collections.singletonList(key), Collections.singletonList(value));
     if (UNLOCK_SUCCESS == var2) {
         return true;
     }
     return false;
 }
```
>>>     首先获取锁对应的value值，检查是否与输入的value相等，如果相等则删除锁（解锁）
>> #### 4.6 Redisson 分布式锁原理
>>> ##### 4.6.1 简介
>>>     Redisson是一个在Redis的基础上实现的Java驻内存数据网格
>>> 代码实现
```java
RLock lock = redisson.getLock("myLock");
lock.lock(); //加锁
lock.unlock(); //解锁
```
>>> ##### 4.6.2 加锁机制
>>>> 加锁流程: 
>>>> <br>![Markdown](https://i.loli.net/2021/04/28/rX4exihgGtLVJDB.png)
>>>>
>>>> Redisson的lock()、tryLock()方法 底层 其实是发送一段lua脚本到一台服务器
```java
if (redis.call('exists' KEYS[1]) == 0) then  //  exists 判断key是否存在
       redis.call('hset' KEYS[1] ARGV[2] 1);  // 如果不存在，hset存哈希表
       redis.call('pexpire' KEYS[1] ARGV[1]);  // 设置过期时间
       return nil;  +                          // 返回null 就是加锁成功
          end;  +
          if (redis.call('hexists' KEYS[1] ARGV[2]) == 1) then  // 如果key存在，查看哈希表中是否存在(当前线程)
              redis.call('hincrby' KEYS[1] ARGV[2] 1);  // 给哈希中的key加1，代表重入1次，以此类推
              redis.call('pexpire' KEYS[1] ARGV[1]);  // 重设过期时间
              return nil;  +
          end;  +
          return redis.call('pttl' KEYS[1]); // 如果前面的if都没进去，说明ARGV[2]的值不同，也就是不是同一线程的锁，这时候直接返回该锁的过期时间
```
>>>> 参数解释：
>>>> <br>KEYS[1]：即加锁的key，RLock lock = redisson.getLock("myLock"); 中的myLock
>>>> <br>ARGV[1]：即 TimeOut 锁key的默认生存时间，默认30秒
>>>> <br>ARGV[2]：代表的是加锁的客户端的ID，类似于这样的：99ead457-bd16-4ec0-81b6-9b7c73546469:1
>>>> <br>其中lock()默认是30秒的生存时间
>>> ##### 4.6.3 锁互斥
>>>     假如客户端A已经拿到了 myLock，现在 有一客户端（未知） 想进入：
>>>     a. 第一个if判断会执行“exists myLock”，发现myLock这个锁key已经存在了
>>>     b. 第二个if判断，判断一下，myLock锁key的hash数据结构中， 如果是客户端A重新请求，证明当前是同一个客户端同一个线程重新进入，所以可从入标志+1，重新刷新生存时间（可重入）； 否则进入下一个if
>>>     c. 第三个if判断，客户端B 会获取到pttl myLock返回的一个数字，这个数字代表了myLock这个锁key的剩余生存时间。比如还剩15000毫秒的生存时间
>>>     此时客户端B会进入一个while循环，不停的尝试加锁
>>> ##### 4.6.4 watch dog 看门狗自动延期机制
>>>     lockWatchdogTimeout（监控锁的看门狗超时，单位：毫秒）默认值：30000
>>>     监控锁的看门狗超时时间单位为毫秒。该参数只适用于分布式锁的加锁请求中未明确使用leaseTimeout参数的情况。(如果设置了leaseTimeout那就会自动失效了呀~)
>>>     看门狗的时间可以自定义设置：config.setLockWatchdogTimeout(30000);
>>>     作用：
>>>        假如客户端A在超时时间内还没执行完毕怎么办呢？ 
>>>        redisson于是提供了这个看门狗，如果还没执行完毕，监听到这个客户端A的线程还持有锁，就去续期，默认是 LockWatchdogTimeout/ 3 即 10 秒监听一次
>>>        如果还持有，就不断的延长锁的有效期（重新给锁设置过期时间，30s）
>>>     可以在lock的参数里面指定：
>>>        lock.lock(); //如果不设置，默认的生存时间是30s，启动看门狗 
>>>        lock.lock(10, TimeUnit.SECONDS);//10秒以后自动解锁，不启动看门狗，锁到期不续
>>>     如果是使用了可重入锁（leaseTimeout）：
>>>        lock.tryLock(); //如果不设置，默认的生存时间是30s，启动看门狗 
>>>        lock.tryLock(100, 10, TimeUnit.SECONDS);//尝试加锁最多等待100秒，上锁以后10秒自动解锁，不启动看门狗
>>> ##### 4.6.5 释放锁机制
>>>     lock.unlock()，就可以释放分布式锁。就是每次都对myLock数据结构中的那个加锁次数减1
>>>     如果发现加锁次数是0了，说明这个客户端已经不再持有锁了，此时就会用：“del myLock”命令，从redis里删除这个key
>>>     为了安全，会先校验是否持有锁再释放，防止
>>>        业务执行还没执行完，锁到期了。（此时没占用锁，再unlock就会报错）
>>>        主线程异常退出、或者假死
```java
finally {
    if (rLock.isLocked()) {
        if (rLock.isHeldByCurrentThread()) {
            rLock.unlock();
        }
    }
}
```
>>> ##### 4.6.6 缺点
>>>     如果是 主从、哨兵模式，当客户端A 把 myLock这个锁 key 的value写入了 master，此时会异步复制给slave实例
>>>     万一在这个主从复制的过程中 master 宕机了，主备切换，slave 变成了master
>>>     那么这个时候 slave还没来得及加锁，此时 客户端A的myLock的 值是没有的，客户端B在请求时，myLock却成功为自己加了锁。这时候分布式锁就失效了，就会导致数据有问题
>>>     所以说Redis分布式说最大的缺点就是宕机导致多个客户端加锁，导致脏数据，不过这种几率还是很小的

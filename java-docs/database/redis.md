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
>> ##### 2.1 String
>>     1. 一个 key 对应一个 value
>>     2. 二进制安全. 可以包含任何数据, 包括图片或者序列化对象
>>     3. value 最多可以是 512M
>>> ##### 2.2 Hash
>>     1. string 类型的 field 和 value 的映射表
>>> ##### 2.3 List
>>     1. 底层是一个链表
>>     2. 按照插入顺序排序, 可以在头部或者尾部插入数据
>>> ##### 2.4 Set
>>     1. string 类型的无序集合, 是通过 HashTable 实现的
>>> ##### 2.5 ZSet (Sorted Set)
>>     1. string 类型的集合
>>     2. 与 Set 不同的是: 每个元素都会关联一个 double 类型的分数
>>     3. Redis 通过分数来为集合中的成员进行从小到大排序
>>     3. zset 成员唯一, 但是分数 (score) 可以重复

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

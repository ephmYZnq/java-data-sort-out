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
>> ##### 2.1 String (字符串)
>> -     一个 key 对应一个 value
>> -     二进制安全. 可以包含任何数据, 包括图片或者序列化对象
>> -     value 最多可以是 512M

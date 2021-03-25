![dict1](../../../../img/middleware/redis-dict-1.png)

![dict2](../../../../img/middleware/redis-dict-2.png)

![dict3](../../../../img/middleware/redis-dict-3.png)
## redisObject
redis 的key-value 结构中的value 是一个redisObject 对象。实际存储类型由encoding 决定。
redisObject 占用16个字节
```
redisObject 对象：string ,list, set, zset, hash
typedef struct redisObject {

    // 类型
    unsigned type:4;

    // 编码
    unsigned encoding:4;

    // 对象最后一次被访问的时间
    unsigned lru:REDIS_LRU_BITS; /* lru time (relative to server.lruclock) */

    // 引用计数
    int refcount;

    // 指向实际值的指针
    void *ptr;

} robj;
```
redis 使用引用计数法管理内存
type :用来客户端约束api 使用的
encoding : redis 底层做的更进一步的优化，val 值的编码。
lru : 关于内存淘汰的
refcount：基于引用计数技术的内存回收机制，当程序不再使用某个对象的时候，这个对象所占用的内存就会被自动释放；另外redis 还通过引用技术实现了对象共享机制。
ptr : 真正指向数据的存储，在64位操作系统中占8字节 64位

help @string 查找所有对string 类型的api 操作

cpu cache line : 缓存行 64字节

bitmap:
setbit key offset 0/1  0(1)级别时间复杂度
offset :2^32 -1 四十多亿

- int: *ptr 直接存储了int 类型的数据。因为int 存储的值不会操作8个字节。
- embstr: cpu cache line : 缓存行 64字节: [redisObject:16字节 + sds:4字节 + 44字节] ，当存储的string 小于44字节时，直接开辟一个64字节的内存空间，将字符串和redisObject放到一起。不需要再进行一次内存io。


现有系统有亿级的活跃用户，如何实现日活统计，为了增强用户粘性，要上线一个连续打卡发送积分的功能，怎么实现连续打卡用户统计。

#### 不同类型和编码的对象
类型|	编码|	对象
---|---|---
String|	int|	整数值实现
String|	embstr|	sds实现 <=39 字节
String|	raw|	sds实现 > 39字节
List|	ziplist|	压缩列表实现
List|	linkedlist|	双端链表实现
Set|	intset|	整数集合使用
Set|	hashtable|	字典实现
Hash|	ziplist|	压缩列表实现
Hash|	hashtable|	字典使用
Sorted set|	ziplist|	压缩列表实现
Sorted set|	skiplist|	跳跃表和字典

#### 多态实现
对一个键执行llen 命令，那么服务器除了要确保执行的命令是列表键外。
1. 如果列表对象编码是 `ziplist` ，那么说明列表对象的实现为压缩列表，程序将使用`ziplist` 函数来返回列表的长度。
2. 如果列表对象的编码是 `linkedlist`，那么说明列表对象的实现是双端链表，程序将用 `linkedlist` 函数来返回双端链表的长度。

#### 内存回收
C语言不具备自动内存回收功能，redisObject 构建一个引用计数技术实现内存回收机制。
refcount ：
- 创建一个新对象时，引用计数值会被初始化为1
- 对象被一个新程序使用时，他的引用计数值会被增一
- 对象不再被一个程序使用时，他的引用计数值会被减一
- 当引用计数值为0 时，对象所占用的内存会被释放

#### 对象共享
redis 初始化服务器时，创建一万个字符串对象，包含0-9999的所有整数。

#### 过期键删除策略
redisDb 结构的 expires 字典保存了数据库中所有键的过期时间--过期字典。过期字典的val 是long 类型的整数存储过期时间。

1. 定时删除：创建一个定时器，让定时器在键的过期时间来临时，立即执行对键的删除操作。
2. 惰性删除：放任键过期不管，但是每次从空间中获取键时，都会检查取得的键是否过期，如果过期就删除该键；没有即返回；
3. 定期删除：每隔一段时间，程序对数据库进行一次检查，删除里面的过期键。

##### 定时删除
对cpu 时间不友好，过期键比较多的情况删除行为可能会占用相当一部分cpu 时间。
##### 惰性删除 
对CPU友好对内存不友好
##### 定期删除
每隔一段时间执行一次删除过期操作，并通过限制删除操作执行的时长和频率减少对CPU时间的影响。
